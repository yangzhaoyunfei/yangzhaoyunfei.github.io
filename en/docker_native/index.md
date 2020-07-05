# CentOS7 安装 Docker


<!--more-->

## centos7 安装docker-ce(社区免费版)
[Reference](https://docs.docker.com/install/linux/docker-ce/centos/)

### 从centos7官方软件仓库安装

#### 使用centos7官方软件仓库
```shell script
# 添加企业linux扩展软件仓库(Extra Packages for Enterprise Linux),并更新仓库缓存
sudo yum install -y epel-release && yum update

# 查看库中dockr软件包信息
yum info docker
```

输出如下:
```text
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
可安装的软件包
名称    ：docker
架构    ：x86_64
时期       ：2
版本    ：1.13.1
发布    ：96.gitb2f74b2.el7.centos
大小    ：18 M
源    ：extras/7/x86_64
简介    ： Automates deployment of containerized applications
网址    ：https://github.com/docker/docker
协议    ： ASL 2.0
描述    ： Docker is an open-source engine that automates the deployment of any
         : application as a lightweight, portable, self-sufficient container that will
         : run virtually anywhere.
         :
         : Docker containers can encapsulate any payload, and will run consistently on
         : and between virtually any server. The same container that a developer builds
         : and tests on a laptop will run at scale, in production*, on VMs, bare-metal
         : servers, OpenStack clusters, public instances, or combinations of the above.
```
***可以看到比较老了, 而且centos7中基本不会更新docker版本了***

```shell script
# 安装docker
sudo yum install -y docker

# 启动 Docker 后台服务
sudo systemctl start docker

# 测试运行 hello-world
sudo docker run hello-world
```

### 从docker官方软件仓库安装(版本较新)

```shell script
# 卸载从centos官方仓库安装的docker
sudo yum autoremove docker -y
```

#### 使用docker官方软件仓库

如果你禁用了默认开启的centos-extras仓库,请重新启用它.

```shell script
# (可选)卸载老版本
sudo yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine

# 安装一些工具软件
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2

# 添加稳定版docker官方软件源
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo

# 更新软件仓库缓存
sudo yum makecache fast

# 安装docker-ce及相关软件
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 启动docker守护进程
sudo systemctl start docker

# 验证安装
sudo docker run hello-world

```

#### 卸载从docker官方仓库安装的docker-ce
```shell script
# 卸载软件
sudo yum remove -y docker-ce

# 删除相关镜像/容器/数据卷
sudo rm -rf /var/lib/docker
```

#### 使用国内的docker镜像仓库(网易镜像)
```shell script
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF

# 重新加载相关信息,重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker

```

#### 以非root 身份管理docker
```shell script
# 添加docker用户组
sudo groupadd docker

# 向用户组添加用户(以foo用户为例)
sudo usermod -aG docker foo

# 注销并重新登录
exit

# 重新登录后, 执行以下命令验证是否生效
docker run hello-world
```

#### 开机启动docker
```shell script
# 设置开机启动
sudo systemctl enable docker

# 禁止开机启动
sudo systemctl disable docker
```

## Debian10 安装docker-ce(社区免费版)
[Reference](https://docs.docker.com/install/linux/docker-ce/debian/)
#### 使用docker官方软件仓库
```shell script
sudo apt-get remove docker docker-engine docker.io containerd runc

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"


sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world
```

## fedora30 安装docker-ce(社区免费版)
[Reference](https://docs.docker.com/install/linux/docker-ce/fedora/)

#### 使用docker官方软件仓库
```shell script
# 卸载老版本
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
# 设置docker官方repository（stable版本）
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo

# 安装docker-ce
sudo dnf install docker-ce docker-ce-cli containerd.io

# 启动docker
sudo systemctl start docker

# 验证安装是否成功
sudo docker run hello-world
```



