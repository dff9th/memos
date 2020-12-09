# Docker install

## Linux kernel update
```
$ sudo -i yum update
```

## Install from the official repository
```
$ sudo -i yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo -i yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo -i yum install -y docker-ce docker-ce-cli containerd.io
```

## Boot
```bash
$ sudo systemctl enable --now docker.service
$ sudo systemctl status docker.service
  docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2020-11-08 04:52:03 UTC; 12s ago
```

## Setup proxy
プロキシ認証設定 (docker pullコマンド等で必要)
```bash
$ sudo mkdir /etc/systemd/system/docker.service.d
$ sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="HTTPS_PROXY=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="FTP_PROXY=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="http_proxy=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="https_proxy=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="ftp_proxy=http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
Environment="NO_PROXY=localhost,127.0.0.1"
```
プロキシ認証設定 (container内で常にexport)
```bash
$ sudo mkdir /root/.docker/config.json
$ sudo vi /root/.docker/config.json
{
  "proxies": {
    "default": {
      "httpProxy": "http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]",
      "httpsProxy": "http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]",
      "ftpProxy": "http://[proxyuser]:[proxypass]@[proxyurl]:[proxyport]"
    }
  }
}
```
反映
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
```
