

# nginx

## centos7安装nginx

~~~shell
# 下载软件包
wget http://nginx.org/download/nginx-1.20.2.tar.gz

# 安装依赖
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

# 解压nginx压缩包
tar -zxvf nginx-1.20.2.tar.gz

#进入nginx目录
cd ./nginx-1.20.2

# 配置nginx prefix：安装目录 安装ssl模块
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

# 编译
make
# 安装
make install
~~~



## 常用命令

~~~shell
#启动
/usr/local/nginx/sbin/nginx

#重新加载配置
/usr/local/nginx/sbin/nginx -s reload

#停止
/usr/local/nginx/sbin/nginx -s stop
~~~

