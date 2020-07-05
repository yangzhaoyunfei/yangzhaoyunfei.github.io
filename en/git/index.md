# Git常用命令


<!--more-->

## 生成ssh-key

```bash
ssh-keygen -t rsa -b 4096 -C "这里输入密钥的注释"

# 留空按三下回车 或 输入密码

# 查看公钥
cat ~/.ssh/id_rsa.pub
```

将公钥添加到 github/gitee/gitlab 之类的平台后，使用ssh协议传输可以不输密码

## 本地git仓库切换 ssh/http 协议
[参考链接](https://blog.csdn.net/yimingsilence/article/details/79980070 )
```bash
# 查看当前remote
git remote -v

# 切换到http：
git remote set-url https://github.com/username/repository.git

# 切换到ssh：
git remote set-url origin git@github.com:username/repository.git
```

## 添加 \ 删除子模块

子模块与父模块使用不同的远程仓库，互不影响,以下命令均在父模块根目录下操作
```shell scriptusername
# 添加
git submodule add -b master https://github.com/username/reponame.git dirname
# 或
git submodule add -b master git@github.com:username/reponame.git dirname

# 移除
git rm submodule-name
```

## 克隆含有子模块的仓库

以下命令均在父模块根目录下操作
```bash
克隆含有子模块的项目, 此时子模块目录是空的
 git clone https://github.com/chaconinc/MainProject

初始化子模块目录为git管理
 git submodule init

拉取子模块文件
 git submodule update
```

## 忽略已跟踪文件

有时候我们添加.gitignore文件之前已经提交过了文件。.gitignore只能忽略那些原来没有被track的文件（自添加以后，从未 add 及 commit 过的文件），如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
```bash
# 删除追踪状态
git rm -r --cached . 

# 把要忽略的文件添加到.gitignore文件后再执行
git add . 
git commit -m "fixed untracked files"
```
在Linux上你可以这样:
```bash
git rm -r --cached . && git add . && git commit -m "fixed untracked files"
```

## 回滚代码到某个commit

```bash
# 回退命令：
git reset --hard HEAD^         回退到上个版本
git reset --hard HEAD~3        回退到前3次提交之前，以此类推，回退到n次提交之前
git reset --hard commit_id     退到/进到 指定commit的sha码

# 强推到远程：
git push origin HEAD --force

```

## 撤销某次提交

我在Git中提交了错误的文件。我该如何撤销那一次commit呢？

```bash
git commit -m "Something terribly misguided"
git reset HEAD~

# << edit files as necessary >>
git add .
git commit -c ORIG_HEAD
```

## 从暂存区 移除/撤回 文件

把文件从暂存区域撤回，但仍然希望保留在当前工作目录中
```bash
git rm --cached README
```

## 清除缓存的用户名和密码

```bash
# 缓存输入的用户名和密码
git config --global credential.helper wincred

# 清除缓存在git中的用户名和密码
git credential-manager uninstall
```

## 仓库fork自别人, 如何在本地合并原作者的更新

```bash
# 1、先克隆项目到本地： 
Git clone https://github.com/iakuf/mojo 
cd mojo 

# 2、添加原作者项目的 remote 地址， 然后将代码 fetch 过来 
git remote add sri https://github.com/kraih/mojo 
git fetch sri 
'sri'相当于一个别名 
# 查看本地项目目录： 
git remote -v 

# 3、合并 
git checkout master 
git merge sri/master 

# 如果有冲突的话，需要丢掉本地分支： 
git reset –hard sri/master 

# 4、这时你的当前本地的项目变成和原作者的主项目一样了，可以把它提交到你的GitHub库 
git commit -am '更新到原作者的主分支' 
git push origin 
git push -u origin master -f # –强制提交

```

## github 上的项目fork自别人, 如何合并原作者的更新

https://blog.csdn.net/qq1332479771/article/details/56087333

## 本地分支关联远程分支

关联前本地仓库中要有远程仓库信息, 没有的要添加:
```bash
git remote remove origin
git remote add origin xxx.git
```

远程库的某分支关联到本地的某分支
```bash
# 常规关联,不指定本地分支就关联到当前所在分支
git branch --set-upstream-to=origin/<branch> dev
# 推送时关联
git push -u origin master
```

## 配置username和email

```bash
# 全局配置
git config --global user.name "唐忠维"
git config --global user.email yangzhaoyunfei@qq.com

# 系统配置
git config --system xxx

# 单个仓库配置
git config --local user.name "唐忠维"
git config --local user.email yangzhaoyunfei@qq.com
```

## 创建仓库
```bash
mkdir aaaa
cd aaaa
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://gitee.com/yangzhaoyunfei/aaaa.git
git push -u origin master
```

```bash
cd existing_git_repo
git remote add origin https://gitee.com/yangzhaoyunfei/aaaa.git
git push -u origin master
```
