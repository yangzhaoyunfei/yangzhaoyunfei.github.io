# Docker 常用命令

<!--more-->
# 常用命令
```shell script
# 启动容器
docker start mysql57

# 停止容器
docker stop mysql57

# 查看运行中容器
docker ps

# 查看所有容器
docker ps -a

# 在运行的容器中运行命令--连接容器,并在其中运行某命令(redis-cli)
docker exec -it redis redis-cli

# 启动docker时启动某容器
docker run mysql57 --restart always

# 从容器拷贝文件到宿主机(在宿主机上执行,无需启动容器)
docker cp mycontainer:/opt/test/file.txt /opt/test/

# 从宿主机拷贝文件到容器(在宿主机上执行,无需启动容器)
docker cp /opt/test/file.txt mycontainer:/opt/test/

```

## 常用选项
-i, --interactive          Attach container's STDIN
-t, --tty                            Allocate a pseudo-TTY
-p
-v


## 推送镜像到repository
```shell script
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```
Make sure to change tagname with your desired image repository tag.
