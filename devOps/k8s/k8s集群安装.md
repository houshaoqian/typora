## 安装k8s(version=v1.23.4)

## 本地环境

宿主机是win10，利用win10的hyper-v搭建了三台centos7虚拟机。

| ip            | 主机名   | 备注         |
| ------------- | -------- | ------------ |
| 172.21.64.101 | centos01 | 主节点master |
| 172.21.64.102 | centos02 | worker节点   |
| 172.21.64.103 | centos03 | worker节点   |



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

在主节点centos01上执行以下命令。其中--kubernetes-version指定k8s版本(上面脚本拉取的镜像)，当未执行上述`pull_k8s_images.sh`脚本时，该命令在尝试拉取指定镜像时会从官方仓库拉取导致超时。
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

密码：hsq12345678910

| Name:     | Default Admin  |
| --------- | -------------- |
| Username: | admin          |
| Type:     | Local          |
| Password  | hsq12345678910 |

