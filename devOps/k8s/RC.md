~~~yaml
# 版本号，例如：v1
apiVersion: v1
# k8s类型，ReplicationController固定值：ReplicationController
kind: ReplicationController
# 元数据的定义
metadata:
  name: rc-tjdk
  # 该Pod隶属的命名空间
  namespace: default
  # 该Pod带有的标签，Map类型
  labels:
    project: tjdk
# Pod规格，对Pod内容的定制
spec:
  # 副本数量
  replicas: 2
  # 当前RC的模板配置
  template:
    metadata:
      name: tjdk
      labels:
        project: tjdk
    # 规格
    spec:
      # 当前RC模板中的容器信息，数组类型
      containers:
      - name: tjdk
        image: tjdk:v1.0
        ports:
        - containerPort: 80
~~~

