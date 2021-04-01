##  redis-单机

拉取镜像

```shell
docker pull redis:latest
```

启动容器

```
docker run -itd --name redis-test -p 6379:6379 redis
```