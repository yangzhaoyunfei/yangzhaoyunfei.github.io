# Linux卸载多余内核


<!--more-->
## Scenarios

在CentOS更新后,并不会自动删除旧内核。所以在启动选项中会有多个内核选项,可以手动使用以下命令删除多余的内核:

## 1.查看系统当前内核版本:

```shell script
uname -a

# 输出如下
Linux localhost.localdomain 3.10.0-229.20.1.el7.x86_64 #1 SMP Tue Nov 3 19:10:07 UTC 201
GNU/Linux
```

## 2.查看系统中全部的内核RPM包:

```shell script
rpm -qa | grep kernel

# 输出如下
kernel-3.10.0-229.14.1.el7.x86_64
```

## 3.删除旧内核的RPM包

```shell script
yum remove kernel-3.10.0-229.14.1.el7
```

## 4.重启系统

```shell script
reboot
```

注意:不需要手动修改/boot/grub/menu.lst

