### 1.安装依赖
  ```
  yum -y install openldap-devel
  yum install pcre pcre-devel -y
  yum -y install openssl openssl-devel
  yum groupinstall "Development Tools" -y
  ```
### 2.下载nginx-auth-ldap模块
  ```
  git clone https://github.com/kvspb/nginx-auth-ldap.git
  ```
### 3.下载tengine压缩包
  ```
  wget http://tengine.taobao.org/download/tengine-2.2.2.tar.gz
  ```
### 4.编译安装tengine
  ```
  tar -xf tengine-2.2.2.tar.gz
  cd tengine-2.2.2
  ./configure
  make && make install
  ```
  tengine默认安装目录为:/usr/loca/nginx.
### 5.编译nginx ldap模块
  假设此处ldap认证模块文件目录为:/usr/local/nginx-auth-ldap.
  ```
  编译nginx ldap认证模块
  cd /usr/loca/nginx/sbin
  ./dso_tool --add-module=/usr/local/nginx-auth-ldap
  ```
### 6.加载nginx ldap模块
  ```
  vim nginx.conf
  events {
    worker_connections  1024;
  }
  dso {
    load ngx_http_auth_ldap_module.so;
  }
  ```
  
