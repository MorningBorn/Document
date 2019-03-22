以往部署vuejs应用都是直接在nginx的location为/下直接部署，此次新项目公用同一域名，要将vue应用部署在/hippo的非根下，使用以往部署方案直接访问就会404。  

部署步骤如下：  

一方面需要开发进行解决，另一方面需要运维解决。  

# 1、开发人员修改项目router配置，如下：
借用别人图片，请将vuejs-admin修改成hippo。    
![图片alt](https://github.com/MorningBorn/Document/blob/master/images/1.png?raw=true)
# 2、开发人员修改build下静态资源加载路径前缀，如下：
借用别人图片，请将vuejs-admin修改成hippo。这里要修改assetsPublicPath为/hippo/地址  
![图片alt](https://github.com/MorningBorn/Document/blob/master/images/1.png?raw=true)


# 3、执行打包命令npm run build，确保所有静态资源均是相对地址/hippo/开头，如下：
借用别人图片，请将vuejs-admin修改成hippo。  
![图片alt](https://github.com/MorningBorn/Document/blob/master/images/3.png?raw=true)


# 4、运维人员修改nginx配置文件，如下：

请必须保证所有代码放在hippo该目录下，才可以正常访问。
```
server {
    listen       80;
    server_name  xxxx.com;
 
    location /vuejs-admin-server {
        proxy_pass http://127.0.0.1:8080/vuejs-admin-server;
    }
    location ^~/hippo {
        alias /code/hippo/;
        try_files $uri $uri/ @rewrites;
    }
    location @rewrites {
        rewrite ^/(hippo)/(.+)$ /$1/index.html last;
    }
}
```
# 5、重新reload nginx即可。
