# Docker 安装 RabbitMQ

<!--more-->
# docker-rabbitmq-latest 安装文档

[参考链接](https://www.runoob.com/docker/docker-install-redis.html)

## 拉取镜像(不加tag则拉取最新版本)
```shell script
docker pull rabbitmq:management

# 查看镜像
docker images rabbitmq:management
```

## 运行容器

### 不指定配置文件
```shell script
docker run -d \
--name rabbitmq \
-p 5672:5672 \
-p 15672:15672 \
rabbitmq:management
```
### 访问管理界面
可以使用默认的账户登录, 用户名和密码都是 ```guest```
http://[宿主机IP]:15672, 如：  
[http://localhost:15672](http://localhost:15672)




