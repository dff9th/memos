# Install GitLab


## 参考
- https://www.gitlab.jp/install/?version=ce#centos-7
- https://docs.gitlab.com/omnibus/settings/nginx.html#manually-configuring-https


## 環境
- VM: GCP
- OS: CentOS7


## Install
Firewall settings
```bash
$ sudo yum install -y curl policycoreutils-python openssh-server
$ sudo systemctl enable sshd
$ sudo systemctl start sshd
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo systemctl reload firewalld
```

For email notification
```bash
$ sudo yum install postfix
$ sudo systemctl enable postfix
$ sudo systemctl start postfix
```

Install GitLab
```bash
$ curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
$ sudo EXTERNAL_URL="https://gitlab.gcp.local" yum install -y gitlab-ce
```


## 自己ルート認証局によって署名されたサーバ証明書の作成・登録
作成は略（ssl-tls.mdを参照）

登録
```
$ sudo cp <証明書> <秘密鍵> /etc/gitlab/ssl/
$ sudo gitlab-ctl restart
```


## クライアント環境のサーバ認証設定
自己ルート証明書をChromeにインポート

DNSにドメイン (gitlab.gcp.local) を追加

Chromeで<https://gitlab.gcp.local>にアクセスし，パスワード設定およびrootユーザでログイン
