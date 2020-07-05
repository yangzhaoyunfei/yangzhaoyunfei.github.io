# CentOS7 安装 MongoDB

<!--more-->
## 配置包管理系统（yum）

创建一个mongodb-org-4.0.repo文件，以便从MongoDB官方软件仓库安装。
```shell script
cat > /etc/yum.repos.d/mongodb-org-4.0.repo << EOF
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
EOF
```

## 安装、配置
```shell script
# 安装最新版
sudo yum install -y mongodb-org

# 配置selinux

# 1.开放27017端口
semanage port -a -t mongod_port_t -p tcp 27017

# 启动
sudo systemctl start mongod.service

# 查看是否启动
tail /var/log/mongodb/mongod.log

# 停止
sudo systemctl stop mongod.service

# 重启
sudo service mongod restart

# 设置开机启动
sudo systemctl enable mongod.service

# 禁止开机启动
sudo systemctl disable mongod.service

## 使用客户端shell连接
mongo --host 127.0.0.1:27017
```

## 使用

mongodb安装好以后，先创建用户管理员，然后才能创建其他用户

### 创建用户管理员(非超级管理员root)

```shell script
# 连接mongo shell
mongo --host 127.0.0.1:27017
```

在mongo-shell输入以下命令

```text
# 切换数据库
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

## 开启安全验证

编辑配置文件
```shell script
vi /etc/mongod.conf
```

找到如下选项, 修改为所示值后保存退出

```text
# 开启身份认证
security:
  authorization: enabled
```

重新启动mongod进程
```shell script
systemctl restart mongod.service
```

## 创建普通用户

```shell script
# 使用用户管理员登录（myUserAdmin用户只具有管理用户和角色的权限可以创建用户）
mongo --port 27017 -u "myUserAdmin" -p '123456' --authenticationDatabase "admin"
```
然后在mongo-shell中输入以下命令
```text
# 先创建/或切换到要创建用户的数据库
use test

# 创建普通用户(需要具有权限的用户操作,如用户管理员,超级管理员)
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
>在上面这个实例中，用户被创建在test数据库上; 该库成为这个用户的身份验证数据库，**用户登陆时需要必须在该库上进行身份验证**;   
>但同时还给该用户分配了，其他数据库（reporting）的角色，使用户在其他数据库上也具有期望的权限。也就是说用户的权限不会局限在身份认证数据库上的
>
## 删除用户
```text
# 删除用户（需要使用具有权限的用户进行操作，比如用户管理员，超级管理员）
use spider
db.dropUser('spider')
```

## 普通用户登录

### 连接时认证

```shell script
mongo --port 27017 -u "myTester" -p "123456" --authenticationDatabase "test"
```

### 连接后认证
认证时, 需要切换到对应的库上, 否则认证会失败
```shell script
mongo --port 27017
```
```text
use test

db.auth("myTester", "123456" )
```
然后就可以在你被授权的数据库上进行一些读写操作了

## 允许远程连接
编辑配置文件
```shell script
vi /etc/mongod.conf
```
找到如下配置项, 并修改为所示值(经测试, 注释该项无效)
```text
bindIp 0.0.0.0 
```
## 卸载
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

## 注意事项

| item      | Description |
| ----------- | ----------- |
| 数据目录       | /var/log/mongodb       |
| 日志目录   | /var/lib/mongo        |
| 配置文件   | /etc/mongod.conf        |

所以需要sudo启动

## 扩展阅读

在数据库中创建的第一个用户应该是具有管理其他用户的权限的用户管理员。

### 什么是认证数据库
添加用户时，可以在特定数据库中创建用户。该数据库是用户的认证的数据库

### 认证用户
要进行身份验证，客户端必须对用户的***身份验证数据库***进行身份验证

#### 权限模型
基于角色的授权方式
认证是验证用户的身份; 授权确定被验证的用户对哪些资源和操作的访问。
各数据库上的角色拥有各种对该库的各种访问权限
用户可以在各个库上担任角色

![pop](/images/middleware/mongodb1.png "image1")


