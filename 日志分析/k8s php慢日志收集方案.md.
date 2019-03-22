# 方案一 filebeat集中收集
php的慢日志通过标准化输出，被docker的log driver引擎进行收集，filebeat通过json log收集输出至es集群中，进行分析。  

+ 优点
日志集中处理  
日志标准化输出之后，可以使用docker的日志轮转策略，避免日志过大  
+ 缺点
php项目众多，多种日志输出，不便于归类和格式化操作  
docker默认的log driver为json file，即将所有的日志行格式化输出为json格式，不方便查看  
由于只有一套filebeat，格式化输出可能会影响其他项目日志分析  
+ 难点
php慢查询日志，在输出时，会主动记录php代码运行的堆栈信息，因此需要修改pod的安全级别  
logstash需要根据多种日志格式创建不通的正则表达式，进行格式化输出  
## 操作流程
### 修改pod的安全级别
可以实现堆栈跟踪。  

```
containers:
- name: php-vipsales-lcs
  image: 192.168.76.172:5000/vipsales-lcs:${BUILD_NUMBER}
  securityContext:
    capabilities:
      add:
      - SYS_PTRACE
```
### 开启php相关日志
修改php配置文件，开启访问日志、慢日志、错误日志，同时重定向到标准输出。  
```
&& sed -i 's#slowlog = log/slow.log#slowlog = /usr/local/var/log/slow_log#g' /usr/local/etc/php-fpm.d/www.conf \\
&& sed -i 's#error_log = /proc/self/fd/2#error_log = /usr/local/var/log/error_log#g' /usr/local/etc/php-fpm.d/www.conf \\
&& sed -i 's#access.log = /proc/self/fd/2#access.log = /usr/local/var/log/access_log#g' /usr/local/etc/php-fpm.d/www.conf \\
&& ln -sf /dev/stdout /usr/local/var/log/access.log \\
&& ln -sf /dev/stderr /usr/local/var/log/error_log \\
&& ln -sf /dev/stderr /usr/local/var/log/slow_log \\
```
通过以上配置，即可实现将pod的php慢日志输出到标准错误输出中。  

之后通过filebeat收集即可。 
# 方案二 pod单独收集
每一pod都单独集成filebeat或者使用sidecar模式，将filebeat集成进pod中，进行单独收集输出。  

+ 优点
可以定制filebeat配置，进行格式化输出
正则表达式不会影响其他的日志收集和输出
+ 缺点
每一pod集成filebeat，会增加pod使用内存大小，同时增加docker镜像的大小
修改filebeat配置，需要修改docker镜像地址
由于日志直接输出在pod内部文件中，可能会造成单个日志文件过大
## 操作流程
### 集成filebeat到php镜像

+ Dockerfile如下：
```
From xxx/xxxx-php-fpm-v1:7.1.24

ENV FILEBEAT_VERSION=6.6.2

COPY ./filebeat.yml /
COPY ./filebeat.tar.gz /
RUN apk add --update-cache curl bash libc6-compat && \
    rm -rf /var/cache/apk/* && \
    cd / && \
    tar xzvf /filebeat.tar.gz && \
    rm /filebeat.tar.gz && \
    mv /filebeat-${FILEBEAT_VERSION}-linux-x86_64 /filebeat && \
    cd /filebeat && \
    cp filebeat /usr/bin && \
    rm -rf /filebeat/filebeat.yml && \
    cp /filebeat.yml /filebeat/ && \
    ls -ltr /filebeat && \
    cat /filebeat/filebeat.yml

WORKDIR /filebeat/
CMD ["./filebeat","-e","-c", "filebeat.yml"]
```
+ filebeat配置文件如下：
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #配置抓取日志文件目录
    - /usr/local/var/log/slow_log
  #配置正则表达式，多行合并
  multiline.pattern: '^\[[0-9]{2}-[a-zA-Z]{3}-[0-9]{4}'
  multiline.negate: true
  multiline.match: after
  #指定输出到kafka的topic名称
  fields:
    log_topics: php-fpm-slow


filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:

output.kafka:
  enabled: true
  hosts: ["192.168.X.X:9092","192.168.X.X:9092","192.168.X.X:9092"]
  #调用配置的kafka topic名称
  topic: '%{[fields][log_topics]}'
  partition.round_robin:
    reachable_only: false
  required_acks: 1
  compression: gzip

  max_message_bytes: 1000000
```

+ docker镜像打包命令如下：
```
docker build -t 192.168.X.X:5000/php-filebeat:v2 .
docker push 192.168.X.X:5000/php-filebeat:v2
```
### 修改jenkins项目配置
修改镜像地址，增加filebeat启动命令。
```
cat > docker-php-entrypoint << EOF
#!/bin/sh
filebeat -e -c /filebeat/filebeat.yml &
nginx -g 'daemon off;' &
php-fpm
EOF
```
### 增加logstash配置

增加对kafka新的topic的消费者。配置如下：  
```
input {
    kafka{
        bootstrap_servers => ["192.168.X.X:9092,192.168.X.X:9092,192.168.X.X:9092"]
        client_id => "phpfpmslow"
        group_id => "phpfpmslow"
        auto_offset_reset => "latest"
        consumer_threads => 5
        topics => ["php-fpm-slow"]
        codec => json
    }
}
output {
    elasticsearch {
        hosts => ["192.168.X.X:9200"]
        index => "%{topics}-%{+YYYY.MM.dd}"
    }
}
```
通过以上操作，即实现了对php慢日志的收集。  

## 结果展示
通过主机名进行匹配。在k8s集群中，主机名包含了服务名，因此通过匹配模式，可进行统计操作。   
```
GET /filebeat-2019.03.22/_search
{
  "query" : {
        "match" : { "host.name" : "lcs" }
    }
}
```
