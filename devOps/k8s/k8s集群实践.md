## 安装k8s(version=v1.23.4)

## 本地环境

宿主机是win10，利用win10的hyper-v搭建了三台centos7虚拟机。

| ip            | 主机名    | 备注 |
| ------------- | --------- | ---- |
| 172.21.64.201 | centos201 |      |
| 172.21.64.202 | centos202 |      |
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



## 生成自签名证书

一键生成ssl自签名证书脚本：
[create_self-signed-cert.sh](./resources/create_self-signed-cert.sh)

生成签名命令

~~~shell
# 创建CA私钥
openssl genrsa -out ca.key 4096

# 自签名机构生成CA证书
openssl req -x509 -new -nodes -sha512 -days 3650 \
-subj "/C=CN/ST=Henan/L=HeBi/O=example/OU=Personal/CN=rancher.k8s" \
-key ca.key \
-out ca.crt

# 客户端私钥证书生成
openssl genrsa -out rancher.k8s.key 4096

openssl req -sha512 -new \
-subj "/C=CN/ST=Henan/L=HeBi/O=example/OU=Personal/CN=rancher.k8s" \
-key rancher.k8s.key \
-out rancher.k8s.csr

# 生成多个域名请求
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=rancher.k8s
DNS.2=harbor.k8s
DNS.3=centos201.k8s
EOF

# 使用自签名CA签发证书
openssl x509 -req -sha512 -days 3650 \
-extfile v3.ext \
-CA ca.crt -CAkey ca.key -CAcreateserial \
-in rancher.k8s.csr \
-out rancher.k8s.crt

~~~



## 安装nginx

参照[centos7上安装nginx](../nginx.md)

ssl证书使用`生成自签名证书`步骤中的输出文件`harbor.k8s.crt` 和 `harbor.k8s.key`。

# Rancher 安装

密码：Abc1234567890

~~~shell
docker run -d \
--privileged \
--restart=unless-stopped   \
--name rancher-rancher \
 -p 8081:80 -p 4431:443  \
rancher/rancher:v2.5.15
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
/usr/local/bin/docker-compose -f /usr/local/harbor/docker-compose.yml up -d
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

