~~~shell
# 查看已安装的软件
dpkg -l #打印所有
dpkg -l | grep <name> #过滤name

# 卸载已安装的程序
dpkg -P <name>#删除安装文件
dpkg -P --purge <name> #删除安装文件且删除配置文件

# apt-get 方式卸载
apt-get remove  --purge <name>


~~~

