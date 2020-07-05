# Svn


<!--more-->

## 概念
* 仓库(版本库)

    仓库是任何一个版本系统的核心，它是开发者们保存工作的总部，仓库不止处理文件还有历史记录，它需要访问网络，扮演**服务器的角色**，版本控制工具扮演**客户端的角色**，客户端可以连接仓库，那么他们就可以从仓库中存储或者提取。仓库通过保存这些更改，一个客户端的更改可以被其他人检索到，一个客户端可以让其他人的更改作为一个工作副本。

* 主干

    trunk 是主要开发所在的目录，经常被项目开发者们查看。

* 标签

    tags 用于标识项目中某个**被命名的快照**，标签操作允许给予对**仓库中特定版本**一个描述和一个难忘的名字。比如，LAST_STABLE_CODE_BEFORE_EMAIL_SUPPORT 比 Repository UUID: 7ceef8cb-3799-40dd-a067-c216ec2e5247 和 Revision: 13 更令人难忘。

* 分支

    分支操作用于创建开发的另一条线，当你想把开发进程复制进两个不同的方向是很有用的。比如，当你发布 5.0 版本时，你可能想从 5.0 的 bug 修复中分离出来创建一个开发 6.0 功能的分支。

* 工作副本( 即本地仓库)

    工作副本是**仓库的一个快照**。这个仓库被所有的成员共享，但人们不直接修改它，相反每个开发者检查这个工作副本，工作副本是一个**私人的工作空间**，这里开发者可以独立于其他成员做自己的工作。

* 提交更改

    提交是一个**保存更改的过程，从私人工作空间提交到中央服务器**。提交后，更改对全部成员可用，通过更新工作副本其他开发者提取这些更改。提交是一个原子操作，要么全部提交成功要么回滚，用户绝不会看到一半完成提交。

* 变更列表

    已被版本控制跟踪的文件, 被修改后默认会被添加到变更列表.

## 在服务端上工作

### 安装svn
```bash
# ubuntu
sudo apt install subversion -y
# centos
sudo dnf install subversion -y
# 测试安装
svn --version
```

### 创建版本库

#### 简单项目
```bash
mkdir /opt/svn
svnadmin create /opt/svn/runoob
```

#### 复杂项目
照惯例，每个 SVN 项目都有主干，标签，分支在项目的 root 目录。
* 主干是主要开发和经常被开发者们查看的目录。
* 分支目录用于追求不同的开发方向。

推荐: 在项目版本库底下创建主干，标签，分支结构。
所以我们采用如下的目录结构:
```text
.
└── svn-template
    ├── branches
    ├── tags
    └── trunk
```
```bash
# 创建典型结构
targetdir=/tmp/svn-template
mkdir $targetdir
svnadmin create $targetdir
mkdir $targetdir/trunk $targetdir/branches $targetdir/tags
```

### 服务端的启动方式
一般都会以多库方式启动
```bash
svnserve -d -r 目录 --listen-port 端口号
```

>-r: 配置方式决定了版本库访问方式。  
--listen-port: 指定SVN监听端口，不加此参数，SVN默认监听3690

* -r直接指定到版本库(称之为单库svnserve方式)
    
    svnserve -d -r /opt/svn/runoob

* 指定到版本库的上级目录(称之为多库svnserve方式)

    svnserve -d -r /opt/svn


## 在客户端上工作

### 检出
```bash
svn checkout svn://192.168.0.1/runoob01 --username=user1
```

### 导入
```bash
# 将某文件夹导入到已有的svn版本库,版本库中abc目录不必存在
svn import -m 'Create trunk, branches, tags directory structure' /xxxdir svn://192.168.0.1/runoob01/abc
```

### 提交
```bash
# 工作
touch readme
# 查看工作副本中的状态, ？说明它还未加到版本控制中; A 表示已添加到变更列表.
svn status
# 添加修改到版本控制; 对已被跟踪的文件的修改会默认添加到变更列表
svn add readme
# 提交到仓库
svn commit -m "SVN readme."

```

### 解决冲突
如对某文件在版本号为100的工作副本上做出变更后, 提交时遇到冲突(仓库中该文件已经是100+版本了), 执行下列命令
```bash
svn update
# 选择mc(以本地的文件为主),做出修改后,再次提交
svn commit -m "change xxx second"
```

### 回退
只对已添加到版本控制的文件有作用
```bash
# M 表示文件已被修改
M       readme
# 回退单个文件
svn revert readme
# 回退整个目录
svn revert -R xxxdir
# 将文件回退到某个版本(即撤销合并;假如现在是22,要退到21)
svn merge -r 22:21 readme 
# 目录同
svn merge -r 22:21 -R xxxdir

# 某版本(22)已提交,但是不想要了, 回退到21,修改后再次提交,即可.
```

### 查看历史消息
* svn log: 用来展示svn 的版本作者、日期、路径等等。
* svn diff: 用来显示特定修改的行级详细信息。
* svn cat: 取得在特定版本的某文件显示在当前屏幕。
* svn list: 显示一个目录或某一版本存在的文件。

[详细用法](https://www.runoob.com/svn/svn-show-history.html)

### 分支
svn的分支,更像是**目录的手动管理**
```bash
# 拷贝主干为一个分支
svn copy trunk/ branches/my_branch
# 查看状态
svn status
# 提交新增的分支到版本库
svn commit -m "add my_branch" 
```

#### 在分支上工作
```bash
cd branches/my_branch/
# 工作.....
```

#### 合并分支
```bash
# 切换回主干分支
cd branches/trunk
# 更新
svn update
# 合并其他分支
svn merge ../branches/my_branch/
# 提交
svn commit -m "xxx"
```

### 标签
使用 tag 的概念，可以给某一个具体版本的代码一个更加有意义的名字。
Tags 即标签主要用于项目开发中的里程碑，比如开发到一定阶段可以单独一个版本作为发布等，它往往代表一个可以固定的完整的版本.
```bash
# 为工作副本创建一个tag
svn copy trunk/ tags/v1.0
# 提交tag内容
svn commit -m "tags v1.0" 
```

## GUI工具
TortoiseSVN  [下载地址](https://tortoisesvn.net/downloads.html), 下载安装;

或者使用如下命令安装
```powershell
choco install tortoisesvn -y
```

## References

* [SVN 教程](https://www.runoob.com/svn/svn-tutorial.html)
