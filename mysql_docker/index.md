# Docker 安装 MySQL

<!--more-->
# docker-mysql-8.x 安装文档

[参考链接](https://www.runoob.com/docker/docker-install-mysql.html)

## 拉取镜像(不加tag则拉取最新版本)
```shell script
docker pull mysql
```

## (可选)在宿主机上创建 将要映射到容器中的目录 及 自定义mysql配置文件.cnf

```shell script
# 数据卷目录
DOCKER_V_MYSQL_DIR=/opt/docker_v/mysql

sudo mkdir -p $DOCKER_V_MYSQL_DIR
sudo chmod 777 -R $DOCKER_V_MYSQL_DIR

mkdir -p $DOCKER_V_MYSQL_DIR/conf
mkdir -p $DOCKER_V_MYSQL_DIR/logs
mkdir -p $DOCKER_V_MYSQL_DIR/data
touch $DOCKER_V_MYSQL_DIR/conf/my.cnf
```

## 运行容器
* 指定数据卷目录的参考5.x,因为该文档实在windows版docker上实践, 故没有使用数据卷

* 不指定数据卷目录
```shell script
docker run -p 3306:3306 \
--name mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d \
mysql
```
## 配置远程登陆
>修改后可能需要刷新权限信息
```shell script
# 进入容器
docker exec -it mysql bash

# 登录mysql
mysql -u root -p

# 修改root密码(可选,推荐)
ALTER USER 'root'@'localhost' IDENTIFIED BY '654321';

# 给root添加远程的登录权限(可选,不推荐)
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';

# 添加远程登录用户(推荐)
CREATE USER 'foo'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'foo'@'%';
```

## 命令说明

```text
-p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口。

-v -v $PWD/conf:/etc/mysql/conf.d：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。

-v $PWD/logs:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs。

-v $PWD/data:/var/lib/mysql ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。

-e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。
```

# docker-mysql-5.x 安装文档

[参考链接](https://www.runoob.com/docker/docker-install-mysql.html)

## 拉取镜像(不加tag则拉取最新版本)
```shell script
docker pull mysql:5.7
```

## (可选)在宿主机上创建 将要映射到容器中的目录 及 自定义mysql配置文件.cnf

```shell script
# 数据卷目录
DOCKER_V_MYSQL_DIR=/opt/docker_v/mysql

sudo mkdir -p $DOCKER_V_MYSQL_DIR
sudo chmod 777 -R $DOCKER_V_MYSQL_DIR

mkdir -p $DOCKER_V_MYSQL_DIR/conf
mkdir -p $DOCKER_V_MYSQL_DIR/logs
mkdir -p $DOCKER_V_MYSQL_DIR/data
touch $DOCKER_V_MYSQL_DIR/conf/my.cnf
```

## 运行容器

```shell script
docker run -p 3306:3306 --name mysql57 \
-v $DOCKER_V_MYSQL_DIR/conf:/etc/mysql/conf.d \
-v $DOCKER_V_MYSQL_DIR/logs:/logs \
-v $DOCKER_V_MYSQL_DIR/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d \
mysql:5.7
```
