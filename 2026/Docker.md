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