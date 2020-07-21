# windows下, 使用choco安装开发环境


<!--more-->

## 安装 chocolatey

管理员运行pwsh, 并在其中运行: 
```shell script
Get-ExecutionPolicy

# If it returns Restricted, then run Set-ExecutionPolicy AllSigned or Set-ExecutionPolicy Bypass -Scope Process

# 执行以下命令并等待安装完成
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# 试用
choco

```

## 使用 choco 安装软件
如果网速较慢，可为powershell设置SS代理 参考 <https://blog.csdn.net/weixin_30553065/article/details/95229332> 
```shell script
$env:HTTPS_PROXY="http://127.0.0.1:1080"
$env:HTTP_PROXY="http://127.0.0.1:1080"
```

新建统一的开发工具管理文件, dev-package.config:
```xml
<?xml version="1.0" encoding="utf-8"?>
    <packages>
      <package id="jdk8" />
      <package id="visualstudio2019community" />
      <package id="adoptopenjdk14" />
      <package id="7zip"/>
      <package id="git" />
      <package id="github-desktop" />
      <package id="hugo-extended" />
      <package id="maven" />
      <package id="gradle" />
      <package id="intellijidea-ultimate" />
      <package id="python" />
      <package id="nodejs" />
      <package id="tomcat" />
      <package id="golang" />
      <package id="vscode" />
      <package id="googlechrome" />
      <package id="docker-desktop" />
      <package id="virtualbox" />
      <package id="teamviewer" />
      <package id="redis-desktop-manager" />
      <package id="studio3t" />
      <package id="heidisql" />
      <package id="sourcetree" />
      <package id="dismplusplus" />
      <package id="tortoisesvn" />
    </packages>
```

安装dev-package.config文件内描述的所有软件包: 
```shell script
choco install dev-package.config -y
```

更多安装包，去这里搜索：<https://chocolatey.org/packages>

## 备份开发环境
创建backup.bat文件，内容如下：

```powershell
REM 备份开发环境

cd %userprofile%
del ./out.zip

tar.exe -a -c -f out.zip ^
.3T ^
.config ^
.docker ^
.dotnet ^
.eclipse ^
.gradle ^
.ideaLibSources ^
.IntelliJIdea* ^
.jmc ^
.m2 ^
.minikube ^
.oss-browser ^
.rdm ^
.ssh ^
.swt ^
.translation ^
.VirtualBox ^
.vscode ^
.vs-kubernetes ^
etc ^
go ^
node_modules ^
PycharmProjects ^
IdeaProjects ^
source ^
tmp ^
'VirtualBox VMs' ^
.gitconfig ^
.viminfo ^
.wslconfig ^
package-lock.json
```

## 恢复开发环境
```powershell
cd ~
Expand-Archive -Path .\out.zip -DestinationPath .
```
## References

* [Windows统一开发环境的基础-Chocolatey](https://zhuanlan.zhihu.com/p/53421288)
