1. Pod
   1. 概念
   2. 分类
   3. 挂载盘Volume + Pod配置项ConfigMap
   4. Pod的生命周期和重启策略
   5. Pod的健康检查
2. Service
   1. 定义
   1. 类型：**ClusterIP**（Service、Headless）、**NodePort**、**LoadBalancer**、**ExternalName**
   2. Headless Service
   3. 集群外部访问Service/Pod
      1. 针对Pod， hostPort或hostNetwork 端口映射到宿主机上；端口唯一，数量限制，ip变动问题。
      2. NodePod，将容器端口映射到k8s集群的端口上，使用任一Node的ip + 该端口号即可访问。端口唯一，数量限制。集群内部已实现负载均衡。
      3. LoadBalancer，云服务器的场景。
      3. Ingress，
   4. DNS服务搭建和配置指南




ConfigMap Deployment Service Pod 
