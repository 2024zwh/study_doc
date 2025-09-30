# Docker 命令与 Dockerfile 参数指南

## Docker 常用命令

### 镜像管理

- **列出所有镜像**
  
  ```bash
  docker images

- 拉取镜像

  

  ```
  docker pull <image_name>:<tag>
  ```

- 推送镜像

  

  ```
  docker push <image_name>:<tag>
  ```

- 构建镜像

  

  ```
  docker build -t <image_name>:<tag> .
  ```

- 删除镜像

  

  ```
  docker rmi <image_id>
  # 强制删除
  docker rmi -f <image_id>
  ```

- 清理悬空镜像

  

  ```
  docker image prune
  ```

### 容器操作

- 运行容器

  

  ```
  docker run [OPTIONS] <image_name>:<tag>
  ```

  - 后台运行：`docker run -d <image_name>:<tag>`
  - 交互模式：`docker run -it <image_name>:<tag> /bin/bash`
  - 端口映射：`docker run -p <host_port>:<container_port> <image_name>:<tag>`
  - 卷挂载：`docker run -v <host_path>:<container_path> <image_name>:<tag>`
  - 指定容器名称：`docker run --name <container_name> <image_name>:<tag>`

- 列出所有容器

  ```
  docker ps
  # 包括停止的容器
  docker ps -a
  ```

- 启动/重启/停止容器

  ```
  docker start <container_id>
  docker restart <container_id>
  docker stop <container_id>
  ```

- 进入运行中的容器

  ```
  docker exec -it <container_id> /bin/bash
  ```

- 查看容器日志

  ```
  docker logs <container_id>
  ```

- 删除容器

  ```
  docker rm <container_id>
  # 强制删除
  docker rm -f <container_id>
  ```

### 其他实用命令

- 查看系统信息

  ```
  docker info
  ```

- 查看容器内部详情

  ```
  docker inspect <container_id>
  ```

## Dockerfile 常用指令

### 基础指令

- FROM

  \- 设置基础镜像

  ```
  FROM <image_name>:<tag>
  ```

- LABEL

  \- 添加元数据

  ```
  LABEL maintainer="user@example.com"
  ```

- RUN

  \- 执行命令并创建新的镜像层

  ```
  RUN apt-get update && apt-get install -y vim
  ```

- CMD

  \- 提供默认执行命令（可以被覆盖）

  ```
  CMD ["executable", "param1", "param2"]
  ```

- ENTRYPOINT

  \- 设置容器启动时要执行的命令（不易被覆盖）

  ```
  ENTRYPOINT ["executable", "param1", "param2"]
  ```

- COPY

  \- 将文件或目录从主机复制到容器

  ```
  COPY <src>... <dest>
  ```

- ADD

  \- 类似于

  ```
  COPY
  ```

  ，但支持 URL 和自动解压 tar 文件

  ```
  ADD <src>... <dest>
  ```

- WORKDIR

  \- 设置工作目录

  ```
  WORKDIR /path/to/workdir
  ```

- EXPOSE

  \- 指定容器将监听的网络端口

  ```
  EXPOSE 80
  ```

- ENV

  \- 设置环境变量

  ```
  ENV MY_VAR=my_value
  ```

- VOLUME

  \- 创建一个挂载点，用于共享数据卷

  ```
  VOLUME ["/data"]
  ```

- USER

  \- 设置运行容器的用户

  ```
  USER daemon
  ```

### 最佳实践

- 使用 `.dockerignore` 文件排除不必要的文件和目录以减少镜像大小。
- 尽量合并多个 `RUN` 指令为一个以减少层数。
- 使用多阶段构建来优化最终镜像大小。