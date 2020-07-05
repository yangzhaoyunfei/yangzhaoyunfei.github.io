# Quick Start

<!--more-->

## 新建一篇文章

1. 进入网站文件夹的根目录。

    ```bash
    cd ~/xxx-blog  # "xxx-blog" 为网站文件夹名。
    ```

1. 使用以下命令新建一篇文章。

    ```bash
    hugo new posts/my-first-post.md  # "my-first-post.md" 是新建文章的文件名。
    ```

1. 编辑新建的文章, 添加内容并保存。

## 本地预览网站效果

1. 启动 Hugo server。

    ```bash
    hugo server
    ```

2. 使用浏览器打开 [http://localhost:1313](http://localhost:1313) 预览。

## 构建网站

1. 在 Hugo 网站文件夹的根目录下, 执行 hugo 命令来构建。

    ```bash
    hugo
    ```

    **注意**：Hugo 会将构建的网站内容默认保存至网站根目录的 public/ 文件夹中。

## 部署网站

1. 将网站根目录下的public/文件夹中的内容发布到你的web服务器
1. 这里是推送到xxx.github.io仓库

## 网站地址

[https://yangzhaoyunfei.github.io](https://yangzhaoyunfei.github.io)


## 自动化部署

1. 在GitHub上创建一个<YOUR-PROJECT>（例如shell-blog）存储库
1. 使用如下命令将创建的hugo项目关联到shell-blog仓库
    ```bash 
    hugo new site shell-bolg  # 创建一个新项目
    ```
1. 启动 hugo server 确认内容无误后, 停止 hugo server, 删除public目录, 重新构建/或创建public目录,并git init 它,创建首次提交,然后使用下面的命令将其添加为子模块.
1. 使用下列命令将hugo 项目下的 public 目录关联到 gitpage的仓库
    ```shell script
    # 这个git子模块将会与hugo项目使用不同的远程仓库, 并互不影响
    git submodule add -b master https://github.com/yangzhaoyunfei/yangzhaoyunfei.github.io.git public
    # 或
    git submodule add -b master git@github.com:yangzhaoyunfei/yangzhaoyunfei.github.io.git public
    ```

1. 在hugo项目根目录（shell-blog/）创建添加自动部署脚本deploy.sh, deploy.bat格式需自行修改
    ```basha
    #!/bin/sh
    
    # If a command fails then the deploy stops
    set -e
    
    printf "Deploying updates to GitHub..." 
    
    # Build the project.
    hugo # if using a theme, replace with `hugo -t <YOURTHEME>`
    
    # Go To Public folder
    cd public
    
    # Add changes to git.
    git add .
    
    # Commit changes.
    msg="rebuilding site $(date)"
    if [ -n "$*" ]; then
        msg="$*"
    fi
    git commit -m "$msg"
    
    # Push source and build repos.
    git push origin master
    ```
    然后您可以运行./deploy.sh "Your optional commit message"将更改发送到<USERNAME>.github.io, 几分钟之后就可以刷新查看了。
    

## 提示
如果删除之后重新添加子模块失败,进行下列操作后重试:
```shell script
# 移除子模块
git rm submodule-name

# 重新克隆远程仓库到public
git clone git@github.com:yangzhaoyunfei/yangzhaoyunfei.github.io.git public

# 再作为子模块管理
git submodule add -b master git@github.com:yangzhaoyunfei/yangzhaoyunfei.github.io.git public
```
