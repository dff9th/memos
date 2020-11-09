# Docker install

## Install from the official repository
```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
```


## Boot
```bash
$ sudo systemctl enable --now docker.service
$ sudo systemctl status docker.service
  docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2020-11-08 04:52:03 UTC; 12s ago
```
