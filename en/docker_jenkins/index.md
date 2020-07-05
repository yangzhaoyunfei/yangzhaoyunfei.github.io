# Docker 安装 jenkins


<!--more-->

```bash
docker run -p 8080:8080 -p 50000:5000 --name jenkins \
-u root \
-v /mydata/jenkins_home:/var/jenkins_home \
-e JENKINS_UC_DOWNLOAD="https://updates.jenkins-zh.cn/update-center.json" \
-d jenkins/jenkins:lts
```

## References 

* [真实有用的 Jenkins 插件安装加速](https://e12e.com/2020/04/06/%E7%9C%9F%E5%AE%9E%E6%9C%89%E7%94%A8%E7%9A%84-Jenkins-%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85%E5%8A%A0%E9%80%9F/)
* [使用Jenkins一键打包部署SpringBoot应用，就是这么6！](https://juejin.im/post/5df780d3e51d4557ff140b30)
* [Jenkins for Docker 跳过插件安装及插件加速镜像设置](https://6xyun.cn/article/92)
