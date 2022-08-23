## 安装k8s(version=v1.23.4)

## 本地环境

宿主机是win10，利用win10的hyper-v搭建了三台centos7虚拟机。

| ip            | 主机名    | 备注 |
| ------------- | --------- | ---- |
| 172.21.64.201 | centos201 |      |
| 172.21.64.202 | centos201 |      |
| 172.21.64.203 | centos203 |      |

三台虚拟主机host

~~~properties
172.21.64.201 rancher.k8s
172.21.64.201 harbor.k8s
172.21.64.201 centos201
172.21.64.202 centos202
172.21.64.203 centos203
~~~

## 安装前准备

~~~shell
# 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

# 关闭SElinux
setenforce 0
sed -i 's/^ *SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

# 禁止swap内存交换
swapoff -a
sed -i.bak '/swap/s/^/#/' /etc/fstab
~~~

## 修改yum源

~~~shell
vim /etc/yum.repos.d/kubernetes.repo
~~~

kubernetes.repo文件内容

~~~txt
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
~~~

更新源

~~~shell
yum clean all
yum -y makecache
~~~



## docker修改镜像地址为国内镜像地址

~~~shell
vim /etc/docker/daemon.json
~~~

daemon.json

其中，"https://harbor.k8s:4431"为虚拟机中安装的harbor私有仓库，因为是自签名证书，因此需要添加到可信任列表中。

~~~json
{
        "insecure-registries":["https://harbor.k8s:4431"],
        "registry-mirrors": [
                "https://o497lg9s.mirror.aliyuncs.com",
                "https://registry.docker-cn.com",
                "https://docker.mirrors.ustc.edu.cn",
                "http://hub-mirror.c.163.com",
                "https://cr.console.aliyun.com/"
        ]
}
~~~

~~~shell
# 重启docker
systemctl daemon-reload
systemctl restart docker
~~~







### 查看k8s可用版本

~~~shell
yum list kubelet --showduplicates
~~~

### 安装工具包

~~~shell
# 安装kubelet
yum install -y kubelet-1.23.4
# 安装kubeadm
yum install -y kubeadm-1.23.4
# 安装kubectl
yum install -y kubectl-1.23.4
~~~

### 启动kubelet服务&设置为开机启动

~~~shell
systemctl enable kubelet && systemctl start kubelet
~~~

### 查看当前k8s依赖的镜像及版本

~~~shell
[root@centos01 ~]# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.23.4
k8s.gcr.io/kube-controller-manager:v1.23.4
k8s.gcr.io/kube-scheduler:v1.23.4
k8s.gcr.io/kube-proxy:v1.23.4
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6

~~~

### 拉取镜像

可以手动`docker pull`上述依赖的镜像版本，或手动使用以下脚本拉取。

~~~shell
#!/bin/bash
url=registry.cn-hangzhou.aliyuncs.com/google_containers
# 安装指定的kubectl版本
version=v1.23.4
# 上面查出来的coredns版本号
coredns=1.8.6
images=(`kubeadm config images list --kubernetes-version=$version|awk -F '/' '{print $2}'`)
for imagename in ${images[@]} ; do
   if [ $imagename = "coredns" ]
   then
      docker pull $url/coredns:$coredns
      # coredns版本前多一个v
      docker tag $url/coredns:$coredns k8s.gcr.io/coredns/coredns:v$coredns
      docker rmi -f $url/coredns:$coredns
   else
      docker pull $url/$imagename
      docker tag $url/$imagename k8s.gcr.io/$imagename
      docker rmi -f $url/$imagename
  fi
done
~~~

将以上脚本保存为脚本`pull_k8s_images.sh`。添加执行权限&执行。

~~~shell
# 添加可执行权限
chmod +x pull_k8s_images.sh

#执行
./pull_k8s_images.sh
~~~



## 初始化集群

### 配置集群网络，指定master节点

在主节点centos102上执行以下命令。其中--kubernetes-version指定k8s版本(上面脚本拉取的镜像)，当未执行上述`pull_k8s_images.sh`脚本时，该命令在尝试拉取指定镜像时会从官方仓库拉取导致超时。
**注意：该命令输出的最后为加该k8s集群的token，需要额外保存。**

~~~shell
# 在主节点上执行
[root@centos01 ~]# kubeadm init --kubernetes-version=1.23.4 --apiserver-advertise-address=172.21.64.101 --pod-network-cidr=10.10.0.0/16
[init] Using Kubernetes version: v1.23.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [centos01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.21.64.101]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [centos01 localhost] and IPs [172.21.64.101 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [centos01 localhost] and IPs [172.21.64.101 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 23.534458 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.23" in namespace kube-system with the configuration for the kubelets in the cluster
NOTE: The "kubelet-config-1.23" naming of the kubelet ConfigMap is deprecated. Once the UnversionedKubeletConfigMap feature gate graduates to Beta the default name will become just "kubelet-config". Kubeadm upgrade will handle this transition transparently.
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node centos01 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node centos01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: ip18kn.2my7txi727usimgk
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.21.64.101:6443 --token ip18kn.2my7txi727usimgk \
        --discovery-token-ca-cert-hash sha256:b791c457ab8416a53839138f7fe7f123611188365affc3b9de9dda2d30b27078

~~~

### 加入worker节点

在机器centos02、centos02节点执行以下命令（token为集群初始化输出的内容），加入k8s集群。

~~~shell
[root@centos02 ~]# systemctl enable kubelet && systemctl start kubelet^C
[root@centos02 ~]# kubeadm join 172.21.64.101:6443 --token ip18kn.2my7txi727usimgk \
>         --discovery-token-ca-cert-hash sha256:b791c457ab8416a53839138f7fe7f123611188365affc3b9de9dda2d30b27078
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
~~~



### 查看集群节点信息

~~~shell
[root@centos01 ~]# kubectl get nodes
NAME       STATUS     ROLES                  AGE     VERSION
centos01   NotReady   control-plane,master   17m     v1.23.4
centos02   NotReady   <none>                 9m10s   v1.23.4
centos03   NotReady   <none>                 8m39s   v1.23.4
~~~

在实际执行过程中，遇到报错：

~~~shell
[root@centos01 ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
~~~

解决方案：

原因是主节点master没有与本机绑定，可通过设置环境变量解决。在集群初始化时怎么避免该问题还不清楚???

~~~shell
# 在配置文件/etc/profile最后一行追加内容
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile

# 使生效
source /etc/profile
~~~

### 安装k8s的网络插件

k8s网络常用的插件有**法兰绒(Flannel)**、**印花布(Calico)**、**针织布(Weave)**、**运河(Canal)**。此处使用了Calico。

~~~shell
# 下载定义文件
[root@centos01 ~]# curl https://docs.projectcalico.org/manifests/calico.yaml

# 创建Calico资源
[root@centos01 ~]# kubectl apply -f calico.yaml 
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/calico-kube-controllers created
~~~

### 查看集群中所有的pods

~~~shell
[root@centos01 ~]# kubectl get pods -o wide -n kube-system
NAME                                       READY   STATUS     RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
calico-kube-controllers-566dc76669-zzcgw   0/1     Pending    0          89s   <none>          <none>     <none>           <none>
calico-node-67lff                          0/1     Init:0/3   0          90s   172.21.64.102   centos02   <none>           <none>
calico-node-ncfkc                          0/1     Init:0/3   0          90s   172.21.64.103   centos03   <none>           <none>
calico-node-pxcx5                          0/1     Init:0/3   0          90s   172.21.64.101   centos01   <none>           <none>
coredns-64897985d-9nfxj                    0/1     Pending    0          26m   <none>          <none>     <none>           <none>
coredns-64897985d-xh6rq                    0/1     Pending    0          26m   <none>          <none>     <none>           <none>
etcd-centos01                              1/1     Running    0          26m   172.21.64.101   centos01   <none>           <none>
kube-apiserver-centos01                    1/1     Running    0          26m   172.21.64.101   centos01   <none>           <none>
kube-controller-manager-centos01           1/1     Running    0          26m   172.21.64.101   centos01   <none>           <none>
kube-proxy-8htl4                           1/1     Running    0          18m   172.21.64.103   centos03   <none>           <none>
kube-proxy-cd6tz                           1/1     Running    0          18m   172.21.64.102   centos02   <none>           <none>
kube-proxy-tlqsn                           1/1     Running    0          26m   172.21.64.101   centos01   <none>           <none>
kube-scheduler-centos01                    1/1     Running    0          26m   172.21.64.101   centos01   <none>           <none>
~~~



## ingress-nginx安装

github地址：[ingress-nginx](https://github.com/kubernetes/ingress-nginx)

[(170条消息) k8s之ingress介绍与实施_江南道人的博客-CSDN博客_ingress 集群](https://blog.csdn.net/huahua1999/article/details/124237412)

# Rancher 安装

密码：Abc1234567890

~~~shell
docker run -d --privileged --restart=unless-stopped \
  --name rancher-rancher \
  -p 8081:80 -p 4431:443 \
  -e NO_PROXY="localhost,127.0.0.1,0.0.0.0,10.0.0.0/8,172.21.64.0/24" \
  -v /etc/docker/rancher/ssl:/ssl/certs \
  -e SSL_CERT_DIR="/ssl/certs" \
  rancher/rancher:latest
~~~



| Name:     | Default Admin |
| --------- | ------------- |
| Username: | admin         |
| Type:     | Local         |
| Password  | Abc1234567890 |



## 搭建harbor私有仓库

创建自签发SSL（使用harbor.k8s访问）

~~~shell
#自签名机构生成CA证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
-subj "/C=CN/ST=Jangsu/L=Nanjing/O=example/OU=Personal/CN=harbor.k8s" \
-key ca.key \
-out ca.crt

#参数说明：
## C，Country，代表国家
## ST，STate，代表省份
## L，Location，代表城市
## O，Organization，代表组织，公司
## OU，Organization Unit，代表部门
## CN，Common Name，代表服务器域名
## emailAddress，代表联系人邮箱地址。

#客户端私钥证书生成
openssl genrsa -out harbor.k8s.key 4096

openssl req -sha512 -new \
-subj "/C=CN/ST=Jangsu/L=Nanjing/O=example/OU=Personal/CN=harbor.k8s" \
-key harbor.k8s.key \
-out harbor.k8s.csr

#生成多个域名请求
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.com
DNS.2=harbor.k8s
DNS.3=harbor.local
EOF

#使用自签名CA签发证书
openssl x509 -req -sha512 -days 3650 \
-extfile v3.ext \
-CA ca.crt -CAkey ca.key -CAcreateserial \
-in harbor.k8s.csr \
-out harbor.k8s.crt

~~~



~~~shell
# 下载安装包
wget https://github.com/goharbor/harbor/releases/download/v2.3.5/harbor-offline-installer-v2.3.5.tgz

# 解压
tar xf harbor-offline-installer-v2.3.5.tgz -C /usr/local/

# 修改默认配置 (主机名称 密码 注释ssl选项)
vim /usr/local/harbor/harbor.yml

# 安装harbor
./usr/local/harbor/prepare
./usr/local/harbor/install.sh

# 设置开启启动，编辑rc.local添加行
vim /etc/rc.local
# 添加行
./usr/local/harbor/docker-compose start
~~~

harbor.yml文件内容

~~~yaml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: 172.21.64.102

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 8081

# https related config
https:
  # https port for harbor, default is 443
  port: 4431
  # The path of cert and key files for nginx
 certificate: /your/certificate/path
 private_key: /your/private/key/path

# # Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal

# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
# external_url: https://reg.mydomain.com:8433

# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: Harbor12345

# Harbor DB configuration
~~~



##  Rancher连接Harbor私有仓库

推荐安装方式： 先安装Rancher再通过Rancher安装Harbor

本次安装的时候，是两者独立安装后，再进行关联的。

在rancher页面上，选中指定 集群 -> 命名空间 -> 执行命令。

~~~shell
# secret-harbor: secret名称，namespace：指定的命名空间
# docker-server：harbor服务地址
# docker-username：harbor登录名
# docker-password：harbor登录密码
kubectl create secret docker-registry secret-harbor --namespace=ingress-nginx \
--docker-server=https://harbor.qiriver.com --docker-username=admin \
--docker-password=Harbor12345 --docker-email=qiriver@163.com
~~~

在rancher中创建服务时，镜像配置指定镜像私服地址和镜像拉取秘钥(命令中创建的secret docker-registry名称 secret-harbor)。

~~~yaml
spec:
    affinity: {}
    containers:
    - image: harbor.qiriver.com/repo/tjdk-harbor:v1.0
    imagePullPolicy: IfNotPresent
    name: container-0
    resources: {}
~~~

