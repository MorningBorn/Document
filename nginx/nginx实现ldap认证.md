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
  ##### tengine默认安装目录为:/usr/loca/nginx


  
