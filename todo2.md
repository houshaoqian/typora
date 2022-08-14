~~~shell
# help -> https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
# 安装kubectl-下载
curl -LO https://dl.k8s.io/release/v1.24.0/bin/linux/amd64/kubectl
# 安装kubectl-安装
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
# 安装kubectl-查看版本
kubectl version --client --output=yaml

# 安装helm-下载
https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz
# 安装helm-解压helm可执行程序到当前目录
tar -zxvf  helm-v3.7.1-linux-amd64.tar.gz -C  .
# 安装helm-将helm移动到$path中
mv helm /usr/local/bin/


helm repo add rancher-<CHART_REPO> http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable

kubectl create namespace cattle-system

docker run -d --privileged --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  rancher/rancher:v2.5.15
~~~

