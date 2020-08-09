# 仓库

仓库类似git的的仓库，用于保存docker镜像。

# 镜像

镜像与容器的区别，类似与类和对象的区别，镜像就是类文件定义，容器就是实例化对象。

1. 查看本地所有的镜像文件

   ~~~shell
   docker images
   ~~~



2. 拉取新的镜像

   docker pull <name>

   ~~~shell
   # 拉去最新版本的redis镜像
   docker pull redis:latest
   ~~~

3. 在仓库中查找镜像

   docker search <name>

   ~~~shell
   # 在仓库中查找redis镜像
   docker search redis
   ~~~



4. 删除本地镜像

   docekr rmi <name>

   ~~~shell
   # 删除本地redis镜像文件
   docker rmi redis
   ~~~

   

# 容器

容器是根据制定镜像文件创建的实例，同一个镜像可以运行出n多容器，容器有name 和 id 属性，name时创建容器时制定，id自动生成。

1. 启动容器

   docker run <options> <imageName>

   options：

   -p：本地端口和容器端口映射

   -v：本地文件映射到容器文件系统

   -i：交互式式操作

   -t：终端

   -d：守护模式，默认启动后会进入容器，加此参数后不会进入容器，容器会以守护进程模式运行。

   ~~~shell
   # 以redis镜像文件为模板，启动容器
   sudo docker run -p 6379:6379 --name redis-signle -v  /data/docker/redis-signle:/redis-signle -d redis redis-server /redis-signle/etc/redis.conf
   ~~~

2. 启动/停止容器

   docker start/stop <容器名称>

   ~~~she
   # 启动redis容器
   docker start redis
   # 停止redis容器
   docker stop redis
   ~~~

3.  查询所有容器

   docker ps -a

4. 删除容器

   docker rm -f <容器名称>

   -f：强制删除，无论容器是否正在运行中

5. 进入正在运行中的容器

   docker -exec [options] <容器名称>

   ~~~she
   docker -exec -it redis
   ~~~

6. 查看容器日志

   docker logs -t -f --tail <行数> <容器名称>

   ~~~she
   docker logs -t -f --tail 50 redis
   ~~~

7. 导入和导出容器

   docker export <容器名称|容器ID>  > <文件路径> 导出的文件即为镜像文件，必须导入到本地镜像库中才能使用

   cat <文件路径> | docker import  - <镜像名称:版本> 中间符号"-"不能省略

   ~~~she
   # 导出容器
   docker export redis>redis.tar
   
   # 导入容器快照，将上述的镜像文件导入到本地镜像库中
   cat redis-signle.tar |sudo docker import - redisbak:1
   ~~~

   