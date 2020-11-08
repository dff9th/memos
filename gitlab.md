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

HTTPS接続用の証明書確認
```bash
$ ls /etc/gitlab/ssl/ -la
合計 12
drwxr-xr-x. 2 root root   98 11月  6 11:29 .
drwxrwxr-x. 4 root root   82 11月  8 02:36 ..
-rw-r--r--. 1 root root 1058 11月  6 11:29 gitlab.gcp.local.crt
-r--------. 1 root root 1675 11月  6 11:29 gitlab.gcp.local.key
-r--------. 1 root root 1679 11月  6 11:29 gitlab.gcp.local.key-staging
```

証明書（.crt）取得およびChromeにインポート（クライアント環境）

DNSにドメイン (gitlab.gcp.local) を追加（クライアント環境）

Chromeで<https://gitlab.gcp.local>にアクセスし，パスワード設定およびrootユーザでログイン
