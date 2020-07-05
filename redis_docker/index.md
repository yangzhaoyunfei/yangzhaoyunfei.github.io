# Docker 安装 Redis

<!--more-->
# docker-redis-latest 安装文档

[参考链接](https://www.runoob.com/docker/docker-install-redis.html)

## 拉取镜像(不加tag则拉取最新版本)
```shell script
docker pull redis

# 查看镜像
docker images redis
```

## (可选)在宿主机上创建 将要映射到容器中的目录 及 自定义mysql配置文件.cnf

```shell script
# 数据卷目录
DOCKER_V_DATA_DIR=/opt/docker_v/redis

sudo mkdir -p $DOCKER_V_DATA_DIR
sudo chmod 777 -R $DOCKER_V_DATA_DIR
mkdir -p $DOCKER_V_DATA_DIR/data
mkdir -p $DOCKER_V_DATA_DIR/conf
touch $DOCKER_V_DATA_DIR/conf/my.cnf
```

## 运行容器
### 不指定配置文件
```shell script
# 如果没有指定数据卷, 这里不要 -v 选项
docker run -d \
-p 6379:6379 \
--name redis_nopasswd \
-v $DOCKER_V_DATA_DIR:/data \
redis:latest \
redis-server --appendonly yes
```
### 指定配置文件
```shell script
# 如果没有指定数据卷, 这里不要 -v 选项
docker run -d \
-p 6379:6379 \
--name redis \
-v $DOCKER_V_DATA_DIR:/data \
-v $DOCKER_V_DATA_DIR/conf/my.cnf:/usr/local/etc/redis/redis.conf \
redis:latest \
redis-server --appendonly yes \
/usr/local/etc/redis/redis.conf
```
## 命令说明

```text
命令说明：
-d : (--detach)脱离shell, 后台运行容器,并打印容器id
-p 6379:6379 : 将容器的6379端口映射到主机的6379端口
-v $DOCKER_V_DATA_DIR/data:/data : 将主机中指定目录下的data挂载到容器的/data
redis-server --appendonly yes : 在容器执行redis-server启动命令, 并打开redis持久化配置
```

## 可视化管理工具
redis-desktop-manager

* [非订阅版下载地址](http://yangzhaoyunfei.oss-cn-beijing.aliyuncs.com/%E5%A4%96%E9%93%BE%E5%88%86%E4%BA%AB%E6%8B%92%E7%BB%9D%E5%88%A0%E9%99%A4/redis-desktop-manager-0.9.3.817.exe)
* [订阅版下载地址](http://yangzhaoyunfei.oss-cn-beijing.aliyuncs.com/%E5%A4%96%E9%93%BE%E5%88%86%E4%BA%AB%E6%8B%92%E7%BB%9D%E5%88%A0%E9%99%A4/redis-desktop-manager-2019.5.176.exe)
