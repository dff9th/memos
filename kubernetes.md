# Kubernetes setup


## CentOS7 on GCP

### 参考
- https://qiita.com/nagase/items/15726e37057e7cc3b8cd
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://github.com/coreos/flannel
- https://metallb.universe.tf/installation/
- https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal
- https://medium.com/digitalfrontiers/kubernetes-ingress-with-nginx-93bdc1ce5fa9
- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://github.com/kubernetes/ingress-nginx/issues/5919

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
$ sudo kubeadm init --apiserver-advertise-address 10.146.0.2 --pod-network-cidr 10.244.0.0/16 --service-cidr 10.96.0.0/12
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

#### k8s-master (In case over WAN access)
Deploy ingress-nginx with NodePort as an ingress controller
```
$ curl -L https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml -o ingress-nginx-controller.yaml
$ kubectl apply -f ingress-nginx-controller.yaml

$ kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-js4xf        0/1     Completed   0          26s
pod/ingress-nginx-admission-patch-llbpb         0/1     Completed   1          26s
pod/ingress-nginx-controller-785557f9c9-4hlll   0/1     Running     0          26s

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.109.153.126   <none>        80:31583/TCP,443:32408/TCP   26s
service/ingress-nginx-controller-admission   ClusterIP   10.106.19.220    <none>        443/TCP                      26s

$ curl localhost:31583
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Deploy sample service
```
$ vi httpd-svc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  containers:
  - name: httpd
    image: httpd
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
  selector:
    app: httpd

$ kubectl apply -f httpd-svc.yaml
```

Deploy sample ingress resource
```
$ vi httpd-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: httpd-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /index
        pathType: Exact
        backend:
          service:
            name: httpd-svc
            port:
              number: 80

$ kubectl apply -f httpd-ingress.yaml
$ curl localhost:31583/index
<html><body><h1>It works!</h1></body></html>
```


### Etc

#### k8s-master (In case over only LAN access)
Deploy metallb as a L4 load balancer in LAN
```
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
$ kubectl get pod -n metallb-system -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
controller-8687cdc65-dtfh4   1/1     Running   0          3m51s   10.244.2.2   k8s-worker1   <none>           <none>
speaker-mnvm6                1/1     Running   0          3m51s   10.146.0.3   k8s-worker1   <none>           <none>
speaker-ndn4s                1/1     Running   0          3m51s   10.146.0.2   k8s-master1   <none>           <none>
speaker-rps7n                1/1     Running   0          3m51s   10.146.0.4   k8s-worker2   <none>           <none>

$ vi metallb-l2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.16.131.1-172.16.131.254

$ kubectl apply -f metallb-l2.yaml
```

Deploy ingress-nginx as an ingress controller
```
$ curl -L https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml -o ingress-nginx-controller.yaml
$ sed -ie 's/NodePort/LoadBalancer/g' ingress-nginx-controller.yaml
$ kubectl apply -f ingress-nginx-controller.yaml

$ kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-p4cgl        0/1     Completed   0          6m18s
pod/ingress-nginx-admission-patch-jhph7         0/1     Completed   0          6m18s
pod/ingress-nginx-controller-785557f9c9-sks9w   1/1     Running     0          6m19s

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.109.167.168   172.16.131.1   80:30343/TCP,443:32491/TCP   6m19s
service/ingress-nginx-controller-admission   ClusterIP      10.101.40.189    <none>         443/TCP                      6m19s

$ curl 172.16.131.1
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```


### Issues

#### Failed to deploy ingress resource (in kubernetes 1.18.0)
```bash
$ vi httpd-ingress.yaml
#apiVersion: networking.k8s.io/v1
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: httpd-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /index
        pathType: Exact
        backend:
          serviceName: "httpd-svc"
          servicePort: 80

$ kubectl apply -f httpd-ingress.yaml
Error from server (InternalError): error when creating "httpd-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s: context deadline exceeded
```

Solve
```bash
$ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
$ kubectl apply -f httpd-ingress.yaml
ingress.networking.k8s.io/httpd-ingress created

$ kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-njc9k        0/1     Completed   0          61m
pod/ingress-nginx-admission-patch-gbfz7         0/1     Completed   0          61m
pod/ingress-nginx-controller-56bbf6d6b7-mfv77   1/1     Running     0          61m

NAME                                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.1.148.187   <none>        80:31251/TCP,443:32023/TCP   61m
service/ingress-nginx-controller-admission   ClusterIP   10.1.48.162    <none>        443/TCP                      61m

$ curl 172.16.12.10:31251/index
<html><body><h1>It works!</h1></body></html>
```

References:
- https://github.com/kubernetes/ingress-nginx/issues/5401
- https://stackoverflow.com/questions/61365202/nginx-ingress-service-ingress-nginx-controller-admission-not-found
