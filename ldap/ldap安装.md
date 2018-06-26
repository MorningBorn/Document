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
