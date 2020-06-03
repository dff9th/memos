# Nginx Install

## CentOS 7
参考: http://nginx.org/en/linux_packages.html#RHEL-CentOS
```
$ sudo -i yum install yum-utils
$ sudo vi /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

$ sudo yum-config-manager --enable nginx-mainline
$ sudo -i yum install nginx
```
