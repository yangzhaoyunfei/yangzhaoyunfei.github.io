# linux下软件多版本管理


<!--more-->

## Scenarios

在linux上我们常常会部署多个软件，从而需要多个运行环境；如有的需要python2，有的需要python3，这是我们就需要进行同一软件的多版本管理。

## Solution

以某版本hugo为例:
```text
opt
└── myapp
    └── hugo
        ├── current -> hugo_0.67.1/
        └── hugo_0.67.1
            ├── LICENSE
            ├── README.md
            └── hugo
```

其中, /opt/myapp 作为可选软件的存放目录,  hugo 为某软件的总目录, hugo_xxx 为某具体版本目录, 多个版本就有多个目录;
如果要将某版本配置到系统PATH变量中, 方便使用, 则为其建立软链接如下: 

```shell script
ln -s hugo_extended_0.67.1_Linux-64bit/ ./current
```

然后再将当前版本的 current/xxx 可执行文件软链接到系统的/usr/local/bin/xxx, 或将软件的current/bin/加到系统path变量

```shell script
sudo ln -s current/hugo /usr/local/bin/
```
或
```shell script
sudo vim /etc/profile

# 添加如下部分
PATH=$PATH:/opt/myapp/golang/current/bin

export PATH
```
