# 1、创建项目目录

在你的服务器上创建一个新目录（例如 `/opt/zentao-docker`）

# 2、并编写配置文件
创建 `docker-compose.yml` 文件，内容如下：

```yaml
version: '3.8'
services:
  zentao:
    image: easysoft/zentao:18.8 # 建议指定一个稳定版本，而非latest
    container_name: zentao_app
    restart: always
    ports:
      - "8080:80" # 将本地8080端口映射到禅道的80端口
    volumes:
      - ./zentao_data:/data # 持久化禅道应用数据
    environment:
      - MYSQL_INTERNAL=false # 关键：不使用内置数据库，改用下面独立的mysql服务
    depends_on:
      - mysql

  mysql:
    image: mysql:5.7 # 使用兼容性较好的MySQL 5.7
    container_name: zentao_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456 # 请设置一个强密码
      MYSQL_DATABASE: zentao # 禅道数据库会自动创建
    volumes:
      - ./mysql_data:/var/lib/mysql # 持久化数据库文件
    command: [
        '--character-set-server=utf8mb4',
        '--collation-server=utf8mb4_unicode_ci'
    ] # 设置字符集，避免中文乱码
```

# 3、启动服务

在存放 `docker-compose.yml` 文件的目录下，执行一条命令即可启动所有服务：

```shell
sudo docker-compose up -d
```

这个命令会拉取镜像（首次运行）并在后台启动禅道和MySQL两个容器。

# 4、访问并安装禅道
等待几十秒容器完全启动后，在本地浏览器访问：`http://你的服务器IP:8080`。

- 首次访问会进入禅道的**网页安装向导**。
- 在数据库配置步骤中，按如下填写：
  - **数据库服务器**：填写 `mysql`（即Compose文件中MySQL服务的容器名称）
  - **数据库用户名**：`root`
  - **数据库密码**：填写你在 `docker-compose.yml` 中设置的密码（例如 `123456`）
  - **数据库名**：`zentao`
- 完成向导后，即可使用初始管理员账号（`admin`，密码 `123456`）登录。

# 5、配置说明

- **数据持久化**：配置中的 `./zentao_data` 和 `./mysql_data` 目录映射，确保了禅道的所有上传文件、代码和数据库数据都保存在本地主机，**即使删除容器，数据也不会丢失**。下次通过 `docker-compose up -d` 即可恢复。
- **资源友好**：本地测试可能资源有限，此配置默认使用较少的资源。如果测试数据量增大，可以在 `docker-compose.yml` 中为 `mysql` 服务添加 `mem_limit: 512m` 之类的限制。
- **版本锁定**：指定了 `easysoft/zentao:18.8` 和 `mysql:5.7` 这样的固定版本，可以确保你的测试环境稳定，不受未来镜像版本升级带来的意外变化影响。
- **关闭服务**：测试完毕后，只需在项目目录下运行 `sudo docker-compose down`。**再次强调，只要不删除本地的 `./zentao_data` 和 `./mysql_data` 文件夹，数据就是安全的。**

# 6、错误排查

1. **检查端口占用**：确保本地服务器的 `8080` 端口没有被其他程序占用。
2. **准备环境**：确保服务器已安装 `Docker` 和 `Docker Compose`。
3. **开始部署**：复制上面的 `docker-compose.yml` 内容，按步骤执行。