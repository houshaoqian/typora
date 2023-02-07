# Netty



**什么是I/O？**

输入输出设备（**I**nput/**O**utput）是对将外部世界信息发送给计算机的设备和将处理结果返回给外部世界的设备的总称。在诸多I/O设备中，作为Java程序猿，我们往往更加关注网卡（即网络I/O）。对网卡的操作，在Java中被封装成了Socket。

> 中间件（redis/MySQL等）开发者，为了提高性能，更加关注对磁盘的操作。



**一次完整网络I/O的流程**

~~~mermaid
sequenceDiagram
Title: 一次网络I/O流程图

participant Client as 客户端
participant Net as 网卡
participant Kernel as 内核
participant Process as 进程

Client ->> Net:①发起请求
Net --x Kernel:②复制数据到内核缓冲区(异步)
Kernel ->> Process:③复制数据到用户进程缓冲区

Note left of Net : 内核通过驱动完成对网卡的读写操作。
Note left of Kernel : 当网卡有数据读入时，通过中断通知操作系统。
Note left of Process : 用户进程的I/O操作，本质是系统调用，切换为内核态后，
Note left of Process : 由操作系统完成数据从内核缓冲区到进程缓冲区的复制。

Process ->> Kernel:④系统调用，写数据到内核缓冲区(异步)
Kernel --x Net:⑤内核态，复制数据到网卡
Net ->> Client:⑥对客户端进行响应


~~~



## 五种I/O模型

​	I/O模型通常根据 **阻塞/非阻塞** 和 **同步/异步** 两个方面进行分类。阻塞/非阻塞是指用户进程在等待读/写时，用户进程是否处于阻塞状态。

同步/异步通常是指在整个I/O流程中，占据主导作用（最终发起I/O请求、或者对数据就绪状态的监控）的是用户进程还是内核进程。

###  一、 同步阻塞I/O

​	

### 二、同步非阻塞



### 三、I/O多路复用



### 四、异步I/O





