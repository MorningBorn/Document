## 前提LDAP已经安装部署完成.
如果使用LDAP仅仅作为用户统一登录中心，则参考安装文档即可；如果ldap要与第三方软件结合，例如confluence、gitlab等结合，则需要开启memberof支持。
请根据实际需要，进行修改使用。

### 1.设置config支持密码验证
  ```
  vim 1-chrootpw.ldif
  
  dn: olcDatabase={0}config,cn=config
  changetype: modify
  add: olcRootPW
  olcRootPW: {SSHA}+qD0AAWp+aUXGlhYIEMr+6ToCqpdNNBB
  ```
### 2.修改相关的domain信息
  ```
  vim 2-chdomain.ldif
  
  dn: olcDatabase={0}config,cn=config
  changetype: modify
  replace: olcRootPW
  olcRootPW: {SSHA}+qD0AAWp+aUXGlhYIEMr+6ToCqpdNNBB
  
  dn: olcDatabase={1}monitor,cn=config
  changetype: modify
  replace: olcAccess
  olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
    read by dn.base="cn=manager,dc=xxx,dc=com" read by * none
  
  dn: olcDatabase={2}hdb,cn=config
  changetype: modify
  replace: olcSuffix
  olcSuffix: dc=xxx,dc=com
  
  dn: olcDatabase={2}hdb,cn=config
  changetype: modify
  replace: olcRootDN
  olcRootDN: cn=manager,dc=xxx,dc=com
  
  dn: olcDatabase={2}hdb,cn=config
  changetype: modify
  replace: olcRootPW
  olcRootPW: {SSHA}+qD0AAWp+aUXGlhYIEMr+6ToCqpdNNBB
  
  dn: olcDatabase={2}hdb,cn=config
  changetype: modify
  replace: olcAccess
  olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=manager,dc=xxx,dc=com" write by anonymous auth by self write by * none
  olcAccess: {1}to dn.base="" by * read
  olcAccess: {2}to * by dn="cn=manager,dc=xxx,dc=com" write by * read
  ```
### 3.开启memberof支持
  ```
  vim 3-load-memberof.ldif
  
  dn: cn=module,cn=config
  objectClass: olcModuleList
  cn: module 
  olcModuleLoad: memberof.la
  olcModulepath: /usr/lib64/openldap
  
  #ldap库路径与操作系统版本是相关的，此处是64位操作系统。
  ```
### 4.新增用户支持memberof配置
  ```
  vim 4-use-memberof.ldif
  
  dn: olcOverlay=memberof,olcDatabase={2}hdb,cn=config
  objectClass: olcConfig
  objectClass: olcMemberOf
  objectClass: olcOverlayConfig
  objectClass: top
  olcOverlay: memberof
  ```
### 5.导入相关配置
  ```
  ldapadd -Y EXTERNAL -H ldapi:/// -f 1-chrootpw.ldif
  ldapadd -Y EXTERNAL -H ldapi:/// -f 2-chdomain.ldif
  ldapadd -Y EXTERNAL -H ldapi:/// -f 3-load-memberof.ldif
  ldapadd -Y EXTERNAL -H ldapi:/// -f 4-use-memberof.ldif
  ```
### 6.查看当前dn下包含cn=config配置列表
  ```
  ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn
  ```
