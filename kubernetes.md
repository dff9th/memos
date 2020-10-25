# Kubernetes setup


## CentOS7 on GCP

### 参考
- https://qiita.com/nagase/items/15726e37057e7cc3b8cd
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://github.com/coreos/flannel

### Environment
- Machines
  - k8s-master1: Kubernetes Master
  - k8s-worker1: Kubernetes Worker
  - k8s-worker2: Kubernetes Worker
- Machine specs
  - all
    - VM: GCP Compute Engine
    - OS: CentOS7
    - CPU: 2Cores
    - Mem: 4GB
  - k8s-master1
    - Internal IP: 10.146.0.2
    - External IP: dynamic
  - k8s-worker1
    - Internal IP: 10.146.0.3
    - External IP: dynamic
  - k8s-worker2
    - Internal IP: 10.146.0.4
    - External IP: dynamic

### Procedure

#### all
Network settings
```bash
$ sudo systemctl disable --now firewalld
$ sudo vi /etc/selinux/config
SELINUX=disabled

$ sudo setenforce 0
$ sudo vi /etc/hosts
10.146.0.2 k8s-master1
10.146.0.3 k8s-worker1
10.146.0.4 k8s-worker2

$ ssh-keygen
$ vi ~/.ssh/authorized_keys
(...Setting public key authentication...)

$ sudo sh -c "cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
"

$ sudo sysctl --system
```

Install docker
```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
$ sudo systemctl enable --now docker
```

Install kubeadm, kubelet, and kubectl
```
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
$ sudo systemctl enable --now kubelet
```

#### k8s-master
Create a cluster with kubeadm
```bash
$ sudo kubeadm init --apiserver-advertise-address 10.146.0.2 --pod-network-cidr 10.244.0.0/16
$ mkdir ~/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl get node
NAME          STATUS   ROLES    AGE   VERSION
k8s-master1   Ready    master   22m   v1.19.3
```

#### k8s-worker{1,2}
Join to the kubernetes cluster
```bash
$ sudo kubeadm join 10.146.0.2:6443 --token <token> \
    --discovery-token-ca-cert-hash <hash>
```

#### k8s-master
Check workers joined
```
$ kubectl get node
NAME          STATUS   ROLES    AGE   VERSION
k8s-master1   Ready    master   29m   v1.19.3
k8s-worker1   Ready    <none>   36s   v1.19.3
k8s-worker2   Ready    <none>   46s   v1.19.3
```
