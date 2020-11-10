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


## Config
Docker outside of Docker用設定 [参考](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding)
```bash
$ sudo vi /etc/gitlab-runner/config.toml 
[[runners]]
  name = "gitlab-runner1"
  url = "https://gitlab.gcp.local/"
  token = "bgCvpdRWhyVx8e8tnik2"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:19.03.13"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
    extra_hosts = ["gitlab.gcp.local:10.146.0.5"]

$ sudo systemctl restart gitlab-runner
```


## プライベートDockerレジストリの設定 (kubernetes master)
GitLabページのSettings > CI/CD > Deploy Tokensでgitlab-deploy-tokenを全権限で作成

.gitlab-ci.yamlで以下を含める
```yaml
  before_script:
    - kubectl create secret docker-registry gitlab-registry --docker-server="$CI_REGISTRY" --docker-username="$CI_DEPLOY_USER" --docker-password="$CI_DEPLOY_PASSWORD" --docker-email="$GITLAB_USER_EMAIL" -o yaml --dry-run | kubectl apply -f -
```
[参考](https://docs.gitlab.com/ee/user/project/clusters/#deployment-variables)
