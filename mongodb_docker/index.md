# Docker 安装 MongoDB

<!--more-->
## 安装, 启动
```shell script
# 拉取镜像, 默认使用latest标签
docker pull mongo

# 查看镜像
docker images mongo

# 启动一个MongoDB容器实例
docker run -itd --name mongo -p 27017:27017 mongo --auth
```

参数说明:
* -p 27017:27017 ：映射容器服务的 27017 端口到宿主机的 27017 端口。
* --auth：开启mongo的安全验证。
* -d: detach, 启动后分离终端, 让容器在后台运行。
* -i: interactive, 交互式操作, 打开容器实例的标准输入, 直到脱离
* -t: tty/teletypewriter, 分配一个模拟终端

## 创建用户管理员

```shell script
# 连接到容器实例, 并运行其中的bash
docker exec -it mongo bash

# 连接mongo-shell(默认参数)
mongo
```
在mongo-shell中进行以下操作
```text
# 切换数据库(默认进入的是test库, 该库上不允许)
use admin

# 创建用户(用户管理员, 用户名/密码 请自行替换)
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "123456",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)

# 退出mongo-shell
exit
```

## 创建普通用户

```shell script
# 连接到容器实例
docker exec -it mongo bash

# 连接mongo-shell(默认参数)
mongo
```
然后在mongo-shell中输入以下命令
```text
# 切换数据库
use admin

# 认证身份(需要切换到对应的库上, 否则认证会失败)
db.auth("myUserAdmin", "123456")

# 创建普通用户(需要具有权限的用户)
db.createUser(
  {
    user: "myTester",
    pwd: "123456",
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)

# 退出 mongo-shell
exit
```

## 删除用户
```text
# 删除用户（需要使用具有权限的用户进行操作，比如用户管理员，超级管理员）
use spider
db.dropUser('spider')
```

## 扩展--使用外部配置文件启动一个MongoDB容器实例
```shell script
docker run --name some-mongo -v /my/custom:/etc/mongo -d mongo --config /etc/mongo/mongod.conf
```

参数说明:
* --config: 指定配置文件路径;
* -v: 映射目录; /my/custom 是容器外部配置文件所在目录; 

由于mongod默认情况下不读取配置文件, 所以指定配置文件路径后, 还需要把外部配置文件所在目录映射到--config指定的目录;


