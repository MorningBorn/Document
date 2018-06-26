## 环境说明
*操作系统：CentOS Linux release 7.5.1804 (Core)  
*LDAP：2.4.44

前提条件：关闭防火墙、selinux，同时进行时钟同步。

### 1.安装软件
  ```
  yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools
  ```
### 2.生成管理员用户和密码
  ```
  slappasswd -s xxxxx
  olcRootPW: {SSHA}+qD0AAWp+aUXGlhYIEMr+6ToCxxxxxx
  需要记录下该密码，后续在配置/etc/openldap/slapd.d/cn=config/olcDatabase\=\{2\}hdb.ldif会使用到。
  ```
### 3.修改域信息、管理员信息
  ```
  vim /etc/openldap/slapd.d/cn=config/olcDatabase\=\{2\}hdb.ldif
  需要修改内容如下：
    olcSuffix: dc=peilian,dc=com    #修改dc名称
    olcRootDN: cn=root,dc=peilian,dc=com    #修改cn名称、dc名称
    olcRootPW: {SSHA}o7XgtJ7XNKKm7qOrtinHqiN7xZ4+gYI9   #该行为新增行，指定管理员密码，该行为新增行
  ```
### 4.修改监控文件管理员信息
  ```
  vim /etc/openldap/slapd.d/cn=config/olcDatabase\=\{1\}monitor.ldif
  原有行内容如下：
    olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=my-domain,dc=com" read by * none
  修改完成内容如下：
    olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=root,dc=peilian,dc=com" read by * none
  ```
### 5.检测ldap配置文件以及版本号
  ```
  检查配置文件。
  slaptest -u
  输出如下：
    5b0502de ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
    5b0502de ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif"
    config file testing succeeded
  表明基本配置文件验证通过。由于这两个文件有crc校验，因此修改完成之后，crc校验失败，会报错，该错误克忽略。
  
  查看版本号。
  slapd -VV
  ```
### 6.启动ldap服务
  ```
  service slapd start
  ```
### 7.配置ldap数据库存储
  ```
  复制基本的数据库配置：
  cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

  修改ldap数据库配置目录所属用户
  chown ldap:ldap -R /var/lib/ldap

  修改ldap数据库配置目录权限
  chmod 700 -R /var/lib/ldap
  ```
