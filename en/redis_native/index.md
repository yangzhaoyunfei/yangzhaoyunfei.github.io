# CentOS7 安装 Redis

<!--more-->
# CentOS7-Redis-4.x 安装

## yum 包管理器安装
>该软件包在 epel-release 需要先安装该软件存储库, centos7.5目前是3.2版本, 不是很新

```shell script
# 安装redis
yum -y install redis
# 启动redis
systemctl restart redis.service

# 查看状态
systemctl status redis.service

# 设置redis开机启动
systemctl enable redis.service

# 测试
redis-cli
```

## 设置redis密码
```shell script
# 编辑配置文件 
sudo  vi /etc/redis.conf
```
找到被注释的# requirepass foobared, 去掉前面的#, 
并把foobared改成你的密码。如: 
```text
# requirepass foobared
```
改为
```text
requirepass 123456
```

## 更改配置文件的位置

redis.conf文件默认在/etc目录下, 你可以更改它的位置和名字  
更改后, 注意在文件```/usr/lib/systemd/system/redis.service```中,   把```ExecStart=/usr/bin/redis-server /etc/redis/6379.conf --daemonize no```中的```xxx.conf```的路径改成的新的路径。


## 编译安装redis-4.x     

### 文件夹准备,(可根据需要自行更改)
```shell script
# 装备安装目录
sudo mkdir /opt/myapp
sudo chown -R yangzhaoyunfei:yangzhaoyunfei /opt/myapp
sudo chmod -R 755 /opt/myapp # 或执行 sudo chmod -R u=rwx,g=rx,o=rx /opt/myapp

# 添加一个目录专门用作自定义path,避免修改系统path
sudo mkdir /opt/path
sudo chown -R yangzhaoyunfei:yangzhaoyunfei /opt/path
sudo chmod -R 755 /opt/path
ll /opt  

# 获取最新的redis稳定版,并解压,查看
wget -c http://download.redis.io/releases/redis-stable.tar.gz && tar -zxf redis-stable.tar.gz && ls

# 安装依赖
sudo yum groupinstall 'Development Tools'
sudo yum install tcl wget

# 编译安装
cd redis-stable/
make distclean && make && make test && make PREFIX=/opt/myapp/redis install

# 创建redis_6379服务,会有一些选项要求选择,配置文件位置也会告诉你/etc/redis/6379.conf
# 备注,脚本添加服务时,redis可执行文件要选择服务端文件: /opt/myapp/redis/bin/redis-server
cd utils
sudo ./install_server.sh

# 启动, 如果要设置密码, 先修改下面的配置文件后在启动
systemctl daemon-reload
sudo systemctl restart redis_6379

# 开机启动
sudo systemctl enable redis_6379

# 创建软链接
sudo ln -s /opt/myapp/redis/bin/redis-server /opt/path/redis-server
sudo ln -s /opt/myapp/redis/bin/redis-cli /opt/path/redis-cli

```


#### 注意 #####
运行配置脚本以后会生成这个文件,使用以下命令打开该文件,进行查看:
```shell script
sudo vi /etc/init.d/redis_6379
```
部分内容如下:
```shell script
EXEC=/opt/myapp/redis/bin/redis-server
CLIEXEC=/opt/myapp/redis/bin/redis-cli
PIDFILE=/var/run/redis_6379.pid
CONF="/etc/redis/6379.conf"
REDISPORT="6379"
AUTH="123456" # 如果设置了密码时要加入这个选项, 不然会导致centos系统启动失败, 启动卡顿等问题
# 还要在如下部分中加上上买了定义的 AUTH 变量
 stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -p $REDISPORT -a $AUTH shutdown
```
