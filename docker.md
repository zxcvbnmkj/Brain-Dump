# Docker
## 介绍
### 推荐阅读
[官方文档](https://docs.docker.com/engine/install/#installation-procedures-for-supported-platforms)
- Docker CE （也叫做 Docker Engine）: 命令行版。[官方下载文档](https://docs.docker.com/engine/install/)
- Docker Desktop : 桌面版，有 GUI 界面，更加推荐。它适用于 Win、MAC、Linux 三种操作系统。[官方下载文档](https://docs.docker.com/desktop/setup/install/windows-install/)
## 安装
### 安装命令行版本（适用于无界面的Linux）
#### centOS & Alibaba Cloud Linux 3（OpenAnolis Edition）
1. 卸载干净，确保无残留
```
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
2. 安装管理软件源所需的插件并设置以后安装的仓库
```
dnf 是新一代的包管理器，代替了 yum
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
因为 https://download.docker.com/ 是外网有点卡，建议配置阿里云镜像
```
sudo dnf config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
3. 安装 docker
```
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
4. 启动 docker 引擎，并自动设置开机自启动
```
sudo systemctl enable --now docker
```
仅启动 docker 但不设置开机自启动
```
sudo systemctl start docker
```
5. 下载测试印象并在容器中运行它
```
sudo docker run hello-world
```
由于网络问题很容易下载失败
```
(base) [root@iZ8vb8tik0yi8f99u87nfbZ ~]# sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'.
```
### 安装桌面版本（对公司收费）
