# Pod

## yaml定义

~~~yaml
# 版本号，例如：v1
apiVersion: v1
# k8s类型，Pod固定值：Pod
kind: Pod
# 元数据的定义
metadata:
  name: string
  # 该Pod隶属的命名空间
  namespace: string
  # 该Pod带有的标签，数组类型
  labels:
    - name: string
  # 该Pod带有的注解标签，数组类型 
  annotations:
    - name: string
# Pod规格，对Pod内容的定制
spec:
  # 对Pod内容器的定制，数组类型，可包含多个容器
  containers:
    # 容器名称，字符串类型，自定义名称
  - name: string
    # 当前容器的启动镜像，字符串类型，必须是仓库中已存在的镜像内容
    image: string
    # 镜像拉取策略，可选值包括Always(默认值)、Never、IfNotPresent
    # Always表示本地有该镜像时仍然重新拉取镜像，Never表示只使用本地镜像
    imagePullPolicy: [Always | Never | IfNotPresent]
    # 容器启动命令列表，数组类型，如果不指定则使用镜像打包时的启动命令
    command: [string]
    # 容器启动参数列表，数组类型
    args: [string]
    # 容器的工作目录，指定容器启动时shell的工作目录，默认为/home
    workingDir: string
    # 挂载盘配置，数组类型，可以挂载多个挂载盘
    volumeMounts:
      # 挂载盘自定义名称
    - name: string
      # 当前挂载盘在容器内Mount的绝对路径，少于512个字符
      mountPath: string
      # 挂载盘读写模式，默认为false，可读可写
      readOnly: boolean
    # 容器需要暴露的端口列表，数组类型
    ports:
      # 自定义的端口名称，
    - name: string
      # 容器需要暴露的端口
      containerPort: int
      # 容器所在主机Node需要暴露的端口，默认是与当前容器暴露端口一致，
      # 由于端口一直，同一台宿主机无法启动第2份副本
      hostPort: int
      # 端口协议，支持TCP和UDP，默认为TCP
      protocol: string
    # 容器需要设置的环境列表，数组类型
    env:
    - name: string
      value: string
    # 当前容器使用资源的限制
    resources:
      # 当前容器最大可使用宿主机主机资源限制
      limits:
        # CPU限制，单位为core数，将用于docker run --cpu-shares参数，
        cpu: string
        # 内存限制，单位可以为[b/k/m/g]，将用于docker run --cpu-memory参数
        memory: string
      # 当前容器启动时，要求宿主主机必须满足的资源
      requests:
        cpu: string
        memory: string
    # 当前容器的健康检查
    livenessProbe:
      # exec方式，即在容器内执行指定命令
      exec:
        # exec方式需要指定的命令或者脚本
        command: [string]
      # http方式，需要指定path、port、scheme等信息 
      httpGet:
        # http方式，指定的path
        path: string
        port: number
        host: string
        scheme: string
        httpHeaders:
        - name: string
          value: string
      # tcp验证方式
      tcpSocket:
        port: number
      # 容器启动后，首次探测时间，单位为秒。
      initialDelaySeconds: 0
      # 探测超时时间
      timeoutSeconds: 0
      # 探测时间间隔，单位秒
      periodSeconds: 0
      # 表示探针的成功的阈值，在达到该次数时，表示成功。默认值为 1，
      # 表示只要成功一次，就算成功了。
      successThreshold: 1
      # 表示探测失败的阈值
      failureThreshold: 1
      securityContext:
        privileged: false
    # Pod的重启策略，Always/Never/OnFailure
    restartPolicy: [Always | Never | OnFailure]
    nodeSelector: object
    imagePullSecrets:
      - name: string
        hostNetwork: false
    # 在该Pod上定义共享挂载盘，数组类型，可定义多个
    volumes:
        # 自定义共享挂载盘名称，容器内引用的挂载盘是在此处定义的。
        # 常见的挂载盘的类型可分为emptyDir/hostPath/nfs/fc/gitRepo/secret等。
      - name: string
        # 类型为emptDir类型的挂载盘，表示与Pod同生命周期的一个临时目录，其值为固定值空对象'{}'。
        emptyDir: {}
        # 类型为hostPath的挂载盘，表示挂载盘是宿主机上的目录，路径由hostPath.path指定。
        hostPath:
          path: string
        # 类型为secret的挂载盘，该类型的挂载盘由k8s集群进行预定义。
        secret:
          # k8s集群中预定义secret挂载盘的引用
          secretName: string
          items:
            - key: string
              path: string
        # # k8s集群中预定义configMap挂载盘的引用
        configMap:
          name: string
          items:
          - key: string
            path: string
~~~



## Pod的类型

### 普通Pod

常见的pod都为该类型，该类型Pod的一旦被创建就会保存在k8s的etcd里。

### 静态Pod

静态Pod并没有保存在k8s的etcd里，而是被存在特定的Node上，并且只在此Node上启动运行。该类型Pod不能通过k8s的API进行管理，无法与ReplicationController、Deployment或者DaemonSet进行关联，并且kubelet无法对他们进行健康检查。静态Pod总是由kubelet创建的，并且总在kubelet所在的Node上运行。

## Pod的生命周期



### Pod的状态

* Pending：API Server已创建该Pod，但在Pod内还有一个或者多个容器的镜像没有，包括正在下载镜像的过程。

* Running：Pod内所有容器均已创建，且至少有一个容器处于运行状态，正在启动状态或正在重启状态。
* Succeeded：Pod内所有容器均成功执行后退出，且不会再重启。

* Failed：Pod内所有容器均已退出，但至少有一个容器退出为失败状态。

* Unknown：由于某种原因，无法获取该Pod状态，可能由于网络等原因。



### Pod的重启策略



