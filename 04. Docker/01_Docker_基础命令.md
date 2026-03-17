# 一、基础命令

## **1、镜像管理 (Images)**

| 命令             | 说明                     | 常用示例                                                 |
| ---------------- | ------------------------ | -------------------------------------------------------- |
| `docker pull`    | 从仓库下载镜像           | `docker pull nginx:latest` `docker pull mysql:8.0`       |
| `docker images`  | 列出本地所有镜像         | `docker images` `docker images | grep nginx (过滤搜索)`  |
| `docker build`   | 根据 Dockerfile 构建镜像 | `docker build -t my-app:v1 .` (`.代表当前目录`)          |
| `docker rmi`     | 删除本地镜像             | `docker rmi <image_id>` `docker rmi -f nginx (强制删除)` |
| `docker tag`     | 给镜像打标签             | `docker tag my-app:v1 user/my-app:v1 (用于推送到私有库)` |
| `docker push`    | 推送镜像到仓库           | `docker push user/my-app:v1`                             |
| `docker history` | 查看镜像构建历史         | `docker history nginx`                                   |

## **2、容器管理 (Containers)**

| 命令             | 说明                 | 常用示例                                                     |
| ---------------- | -------------------- | ------------------------------------------------------------ |
| `docker run`     | 创建并启动一个新容器 | `docker run -d -p 80:80 --name web nginx`<br /> `-d: 后台运行 -p: 端口映射 --name: 指定名称` |
| `docker ps`      | 列出正在运行的容器   | `docker ps` <br />`docker ps -a (列出所有，包括已停止的)`    |
| `docker start`   | 启动已停止的容器     | `docker start <container_id>`                                |
| `docker stop`    | 优雅停止容器         | `docker stop <container_id>`                                 |
| `docker restart` | 重启容器             | `docker restart <container_id>`                              |
| `docker rm`      | 删除已停止的容器     | `docker rm <container_id>` <br />`docker rm -f <container_id> (强制删除运行中的)` |
| `docker rename`  | 重命名容器           | `docker rename old_name new_name`                            |

## **3、调试与交互 (Debug & Interaction)**

| 命令             | 说明                          | 常用示例                                                     |
| ---------------- | ----------------------------- | ------------------------------------------------------------ |
| `docker logs`    | 查看容器日志                  | `docker logs -f <container_id> (实时跟踪)`  <br />`docker logs --tail 100 <id> (只看最后100行)` |
| `docker exec`    | 在运行中的容器内执行命令      | `docker exec -it <container_id> bash (进入交互终端)`  <br />`docker exec <id> ls /app` |
| `docker inspect` | 查看容器/镜像的详细信息(JSON) | `docker inspect <container_id>` <br />`docker inspect --format='{{.NetworkSettings.IPAddress}}' <id> (只查IP)` |
| `docker top`     | 查看容器内的进程              | `docker top <container_id>`                                  |
| `docker stats`   | 实时查看资源占用 (CPU/内存)   | `docker stats (类似 Linux 的 top 命令)`                      |
| `docker diff`    | 查看容器文件系统的变更        | `docker diff <container_id>`                                 |

## **4、清理与维护 (System & Cleanup)**

| 命令                     | 说明                       | 常用示例                                                     |
| ------------------------ | -------------------------- | ------------------------------------------------------------ |
| `docker system df`       | 查看 Docker 磁盘使用情况   | `docker system df`                                           |
| `docker container prune` | 删除所有已停止的容器       | `docker container prune`                                     |
| `docker image prune`     | 删除所有悬空镜像           | `docker image prune`                                         |
| `docker system prune`    | 大招：清理所有未使用的资源 | `docker system prune -a  (!慎用：会删除所有未运行容器的镜像和网络)` |
| `docker info`            | 查看系统级信息             | `docker info (查看存储驱动、容器数量等)`                     |

## **5、网络与数据卷 (Network & Volumes)**

| 命令                     | 说明             | 常用示例                                       |
| ------------------------ | ---------------- | ---------------------------------------------- |
| `docker volume create`   | 创建数据卷       | `docker volume create db-data`                 |
| `docker volume ls`       | 列出数据卷       | `docker volume ls`                             |
| `docker network create`  | 创建自定义网络   | `docker network create my-net`                 |
| `docker network connect` | 将容器连接到网络 | `docker network connect my-net <container_id>` |

## **6、Docker Compose (编排神器)**

| 命令                   | 说明                      | 常用示例                                  |
| ---------------------- | ------------------------- | ----------------------------------------- |
| `docker-compose up`    | 启动所有服务              | `docker-compose up -d (后台运行)`         |
| `docker-compose down`  | 停止并移除容器、网络      | `docker-compose down -v (同时删除数据卷)` |
| `docker-compose ps`    | 查看Compose管理的容器状态 | `docker-compose ps`                       |
| `docker-compose logs`  | 查看所有服务日志          | `docker-compose logs -f [service_name]`   |
| `docker-compose build` | 重新构建镜像              | `docker-compose build --no-cache`         |

*(注：新版 Docker 已支持 `docker compose` 作为原生插件，去掉了连字符，用法相同)*

# 二、文件操作

## **1、宿主机>容器**

```dockerfile
# 语法：docker cp <宿主机路径> <容器名/ID>:<容器内路径>

# 示例：把本地的 config.json 复制到容器的 /app 目录下
docker cp ./config.json my_container:/app/

# 示例：把本地整个文件夹复制到容器
docker cp ./logs/ my_container:/var/logs/
```

## **2、容器>宿主机**

```dockerfile
# 语法：docker cp <容器名/ID>:<容器内路径> <宿主机路径>

# 示例：把容器里的日志文件拷出来查看
docker cp my_container:/var/log/app.log ./local_log.log

# 示例：把容器整个目录拷出来备份
docker cp my_container:/data/ ./backup_data/
```

## **3、容器内部**

```dockerfile
# 推荐方式：启动一个交互式的 bash shell
docker exec -it <容器名/ID> bash

# 如果容器里没有 bash (如 Alpine 镜像)，尝试 sh
docker exec -it <容器名/ID> sh
```

```dockerfile
进入容器后，你就是 root 用户（默认），可以执行：
查看文件：ls -l, cat filename, less filename
编辑文件：vi filename 或 nano filename (如果镜像里装了编辑器)
删除文件：rm filename 或 rm -rf foldername
移动/重命名：mv old_name new_name
创建目录：mkdir new_folder
⚠️ 注意：在容器内直接修改文件是临时的。如果容器被删除重建，这些修改会丢失（除非你挂载了 Volume 数据卷）。
```

# 三、**核心总结与最佳实践**

## **1. 记忆口诀**

- **查状态**：`ps`, `images`, `stats`
- **看日志**：`logs -f`
- **进容器**：`exec -it ... bash`
- **跑服务**：`run -d -p`
- **清垃圾**：`system prune`

## **2. 常用参数速记**

- `-d` (Detach): 后台运行，不占用当前终端。
- `-it` (Interactive + TTY): 交互式终端，用于进入容器内部。
- `-p` (Port): 端口映射 (`宿主机端口:容器端口`)。
- `-v` (Volume): 挂载数据卷 (`宿主机路径:容器路径`)，**数据持久化关键**。
- `--name`: 给容器起个好记的名字，不然每次都要输那一长串 ID。
- `--rm`: 容器停止后自动删除，适合一次性任务。
- `--network`: 指定网络，实现容器间通过名字互相访问。

## **3. 避坑指南**

- **不要直接在运行中的容器里修改代码**：容器是“不可变基础设施”。如果需要修改，应该修改源代码 -> 重新 `build` 镜像 -> 重新 `run` 容器。直接在容器里改，下次重启就没了。
- **数据持久化**：数据库、配置文件等重要数据，**必须**使用 `-v` 挂载到宿主机或使用 Volume，否则容器删除后数据会丢失。
- **资源限制**：生产环境建议使用 `--memory` 和 `--cpus` 限制容器的资源占用，防止单个容器拖垮宿主机。
- **清理习惯**：定期运行 `docker system prune`，开发过程中会产生大量中间镜像和停止的容器，很容易占满磁盘。

## **4. 现代工作流建议**

对于复杂的应用（如微服务、数据库+后端+前端），**不要**单独使用 `docker run` 命令，而是编写 `docker-compose.yml` 文件。

- **旧方式**：手动敲一堆 `docker run -p ... -v ... --link ...`，容易出错且难维护。
- **新方式**：`docker compose up -d`，配置即代码，一键启停整个环境。

这份清单涵盖了 95% 的日常需求。你可以将此页面收藏，遇到具体问题时快速查找对应的命令。







