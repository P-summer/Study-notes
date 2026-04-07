# Docker 常用命令速查表

## 1. 镜像操作

| 命令 | 说明 |
|------|------|
| `docker images` | 列出本地所有镜像 |
| `docker search <镜像名>` | 在 Docker Hub 搜索镜像 |
| `docker pull <镜像名>:<标签>` | 拉取镜像（标签默认 latest） |
| `docker push <仓库>/<镜像名>:<标签>` | 推送镜像到远程仓库 |
| `docker rmi <镜像ID或名称>` | 删除本地镜像（加 `-f` 强制） |
| `docker save -o <文件名>.tar <镜像名>` | 导出镜像为 tar 文件 |
| `docker load -i <文件名>.tar` | 从 tar 文件导入镜像 |
| `docker tag <镜像ID> <新标签>` | 给镜像打标签 |
| `docker history <镜像名>` | 查看镜像构建历史 |
| `docker image prune` | 删除所有未使用的镜像 |

## 2. 容器操作

| 命令 | 说明 |
|------|------|
| `docker run <镜像名>` | 创建并启动容器 |
| `docker run -it <镜像名> /bin/bash` | 以交互模式运行并进入容器 |
| `docker run -d --name <容器名> <镜像名>` | 后台运行并指定容器名 |
| `docker run -v <宿主机路径>:<容器路径> <镜像名>` | 挂载目录（数据持久化） |
| `docker run -p <宿主机端口>:<容器端口> <镜像名>` | 端口映射 |
| `docker ps` | 列出运行中的容器 |
| `docker ps -a` | 列出所有容器（含已停止） |
| `docker stop <容器ID或名称>` | 停止容器 |
| `docker start <容器ID或名称>` | 启动已停止的容器 |
| `docker restart <容器ID或名称>` | 重启容器 |
| `docker rm <容器ID或名称>` | 删除容器（加 `-f` 强制删除运行中的） |
| `docker rm $(docker ps -aq)` | 删除所有已停止的容器 |
| `docker exec -it <容器名> /bin/bash` | 进入正在运行的容器 |
| `docker logs <容器名>` | 查看容器日志（加 `-f` 实时跟踪） |
| `docker inspect <容器名>` | 查看容器详细信息 |
| `docker top <容器名>` | 查看容器内进程 |
| `docker cp <本地文件> <容器名>:<容器路径>` | 复制文件到容器 |
| `docker cp <容器名>:<容器路径> <本地路径>` | 从容器复制文件到宿主机 |
| `docker rename <旧名> <新名>` | 重命名容器 |
| `docker container prune` | 删除所有已停止的容器 |

## 3. 数据卷（Volume）

| 命令 | 说明 |
|------|------|
| `docker volume ls` | 列出所有数据卷 |
| `docker volume create <卷名>` | 创建数据卷 |
| `docker volume inspect <卷名>` | 查看数据卷详情 |
| `docker volume rm <卷名>` | 删除数据卷 |
| `docker volume prune` | 删除所有未使用的数据卷 |
| `docker run -v <卷名>:<容器路径> <镜像名>` | 挂载数据卷 |

## 4. 网络操作

| 命令 | 说明 |
|------|------|
| `docker network ls` | 列出所有网络 |
| `docker network create <网络名>` | 创建自定义网络 |
| `docker network inspect <网络名>` | 查看网络详情 |
| `docker network connect <网络名> <容器名>` | 将容器连接到网络 |
| `docker network disconnect <网络名> <容器名>` | 断开容器网络 |
| `docker network rm <网络名>` | 删除网络 |
| `docker network prune` | 删除所有未使用的网络 |

## 5. 系统与资源管理

| 命令 | 说明 |
|------|------|
| `docker version` | 显示 Docker 版本信息 |
| `docker info` | 显示 Docker 系统信息 |
| `docker stats` | 实时查看容器资源使用情况 |
| `docker system df` | 查看 Docker 磁盘使用情况 |
| `docker system prune -a` | 清理所有未使用的容器、镜像、网络、构建缓存（危险，谨慎使用） |
| `docker events` | 实时监控 Docker 事件 |

## 6. Docker Compose（多容器编排）

| 命令 | 说明 |
|------|------|
| `docker-compose up` | 启动所有服务（加 `-d` 后台运行） |
| `docker-compose down` | 停止并删除所有容器、网络 |
| `docker-compose down -v` | 同时删除数据卷 |
| `docker-compose ps` | 查看服务状态 |
| `docker-compose logs` | 查看日志 |
| `docker-compose exec <服务名> <命令>` | 在运行中的服务容器内执行命令 |
| `docker-compose build` | 重新构建镜像 |
| `docker-compose pull` | 拉取所有服务的最新镜像 |
| `docker-compose restart` | 重启所有服务 |
| `docker-compose stop` | 停止所有服务 |
| `docker-compose start` | 启动已存在的服务 |

## 7. 常用组合技巧

```bash
# 停止并删除所有运行中的容器
docker stop $(docker ps -q) && docker rm $(docker ps -aq)

# 删除所有 `<none>` 镜像（虚悬镜像）
docker rmi $(docker images -f "dangling=true" -q)

# 进入容器的 shell（适用于 alpine 或 bash 环境）
docker exec -it <容器名> sh

# 查看容器重启次数
docker inspect -f "{{ .RestartCount }}" <容器名>

# 实时查看容器日志并显示时间戳
docker logs -ft <容器名>
