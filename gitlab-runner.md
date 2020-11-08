# GitLab Runner install

## Insall from the official repository
```bash
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
$ sudo yum install gitlab-runner
```


## 自己証明書を用いたHTTPS接続の設定
```bash
$ sudo mkdir /etc/gitlab-runner/certs
$ sudo cp <GitLabの自己証明書> /etc/gitlab-runner/certs/
```

## Register
```bash
$ sudo gitlab-runner register
```
