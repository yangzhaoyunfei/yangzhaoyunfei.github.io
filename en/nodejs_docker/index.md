# Docker 安装 NodeJS

<!--more-->
```bash
# 拉取镜像
docker pull node:lts

# 查看镜像
docker images node

# 运行容器
docker run -itd --name mynode node:lts

# 连接到容器
docker exec -it mynode /bin/bash

# 参看版本
node -v
```

