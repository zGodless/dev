#
 ## 一个部署运行命令：docker run -d --name webgo -p 8080:8080 -v /app/web1:/app web1-app
    解析命令：运行webgo这个镜像，并将容器的8080端口映射到宿主机的8080端口，
            同时使用挂载卷将容器的/app路径映射到宿主机的/app/web1路径下。
1、Docker是软件部署技术，使用容器部署
2、与虚拟机的区别：docker所有容器共用一个系统内核，而虚拟机每个都包含一个完整独立的操作系统，docker更轻量
3、docker镜像仓库：hub.docker.com
4、阿里云的系统镜像无法直接从官网安装docker,需要从国内镜像源安装
    先安装yum-utils  执行命令：sudo dnf install -y yum-utils

    接下来，我们将使用 dnf config-manager 添加阿里云的 Docker CE 官方 YUM 源。 阿里云为 Docker 提供了国内镜像源，地址为：https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
    执行以下命令添加 Docker 镜像源：sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
    Docker 官方 YUM 源已成功添加。接下来，由于 Alibaba Cloud Linux 3 基于 RHEL 8 兼容体系，我们需要安装一个适配插件 dnf-plugin-releasever-adapter，以确保系统能正确识别仓库中的软件包版本。 
    执行以下命令安装该适配插件：sudo dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
    适配插件已安装。现在我们可以直接使用 dnf 安装 Docker。 
    执行以下命令安装最新版的 Docker CE（社区版）：sudo dnf -y install docker-ce docker-ce-cli containerd.io

    Docker 已成功安装！当前安装的版本为 Docker CE 26.1.3，并已自动包含 docker-compose 插件（可通过 docker compose 命令使用）。 接下来您可以：
    启动 Docker 服务：sudo systemctl start docker
    设置开机自启：sudo systemctl enable docker
    验证安装：docker --version 和 docker run hello-world
    查看镜像列表：docker images
    查看进程状态：docker ps(process status)
    创建并运行一个容器：docker run
    容器启动：docker start
    容器停止：docker stop

 ## docker compose
    Docker Compose 可以理解成：是单机服务器上管理多容器应用的“配置化启动工具”。用一个 YAML 配置文件，把一组 Docker 容器定义、启动、连接和管理起来。
如果说 docker run ... 是手动启动一个容器，那么 Docker Compose 就是把很多条 docker run 命令整理成一个 compose.yaml 文件，然后用一条命令启动整套服务。

 ### Docker Compose 里最重要的几个概念是：
概念	        说明
services	    服务，也就是要运行的容器定义
image	        使用哪个镜像
build	        从本地 Dockerfile 构建镜像
ports	        端口映射
volumes	        数据卷或目录挂载
environment	    环境变量
env_file	    从 .env 文件读取环境变量
networks	    容器网络
depends_on	    服务启动顺序
restart	        容器退出后的重启策略
healthcheck	    健康检查

 ### 一个典型结构是：
services:
  服务名:
    image: 镜像名
    build: 构建路径
    container_name: 容器名
    ports:
      - "宿主机端口:容器端口"
    volumes:
      - 宿主机路径或数据卷:容器路径
    environment:
      KEY: value
    env_file:
      - .env
    depends_on:
      - 另一个服务名
    restart: unless-stopped
    networks:
      - 网络名

volumes:
  数据卷名:

networks:
  网络名:
例如：
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
含义是：
使用 nginx:latest 镜像
把服务器的 80 端口映射到容器的 80 端口
把当前目录下的 html 挂载到 Nginx 默认站点目录
容器异常退出后自动重启

### 容器本身是临时的，删除容器后，容器内部的数据也可能丢失。所以数据库、上传文件、持久化配置一般都要用 volumes。

### 服务之间怎么通信
Compose 会自动给这一组服务创建一个内部网络。

### .env 文件
Compose 支持 .env 文件。
例如 
.env：
    POSTGRES_USER=myuser
    POSTGRES_PASSWORD=mypassword
    POSTGRES_DB=myapp
    APP_PORT=3000

compose.yaml：
    services:
    db:
        image: postgres:16
        environment:
        POSTGRES_USER: ${POSTGRES_USER}
        POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
        POSTGRES_DB: ${POSTGRES_DB}

    app:
        image: myapp
        ports:
        - "${APP_PORT}:3000"
好处是：
密码、端口、域名等配置可以从主配置里分离
同一个 Compose 文件可以用于不同环境
更新配置比较方便
不过 .env 里如果有密码，不要随便提交到 Git 仓库。

 ### docker compose适合小项目、测试环境
 不太适合哪些场景
    Docker Compose 不太适合直接管理大规模集群。
    如果你的需求是：
    多台服务器自动调度
    服务自动扩缩容
    滚动发布
    高可用编排
    自动服务发现
    复杂权限和网络策略
    通常会考虑 Kubernetes、Docker Swarm、Nomad 之类的系统。
    但对于很多个人项目、小团队项目、单机服务，Compose 已经非常够用。

 ## Docker Swarm 集群工具
    初始化：docker swarm init