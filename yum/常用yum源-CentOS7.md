### 1.阿里云epel源
  ```
  [epel]
  name=Extra Packages for Enterprise Linux 7 - $basearch
  baseurl=http://mirrors.aliyun.com/epel/7/$basearch
  failovermethod=priority
  enabled=1
  gpgcheck=0
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
  ```
### 2.php7安装源
  ```
  [webtatic]
  name=Webtatic Repository EL7 - $basearch
  mirrorlist=http://mirror.webtatic.com/yum/el7/$basearch/mirrorlist
  failovermethod=priority
  enabled=1
  gpgcheck=0
  ```
