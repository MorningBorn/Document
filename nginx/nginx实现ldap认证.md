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
### 7.配置nginx ldap认证
  ###### 以下两段配置均在http段进行配置。
  配置ldap认证服务器信息.
  ```
  ldap_server xxx-ldap {
      url ldap://ldap服务器ip:port/DC=xxx,DC=com?cn?sub?(objectClass=person);
      binddn "cn=xxx,dc=xxx,dc=xxx";
      binddn_passwd "binddn密码";
      group_attribute uniquemember;
      group_attribute_is_dn on;
      require valid_user;
  }
  ```
  配置location段使用ldap认证.
  ```
  server {
    listen       80;
    server_name  localhost;
    location /status {
      stub_status on;
      auth_ldap "Forbidden";
      auth_ldap_servers xxx-ldap;
  }
  ```
  ###### 此处ldap认证使用cn作为用户名进行认证。
  
