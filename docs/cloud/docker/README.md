# Docker基础

## 虚拟化技术

### 概述

xxx

### 特点

- 基础镜像 GB 级别。
- 创建使用稍微复杂。
- 隔离性强。
- 启动速度慢。
- 移植与分享不方便。

## 容器化技术

### 概述

xx

### 特点

- 基础镜像MB级别。
- 创建简单。
- 隔离性强。
- 启动速度秒级。
- 移植与分享方便。

## 解决问题

### 统一标准

- **应用构建** 不同的编程语言的都构建成镜像交付。
- **应用分享** 通过仓库统一管理镜像。
- **应用运行** 通过统一的运行命令运行镜像。

### 资源隔离

- cpu、memory资源隔离与限制。
- 访问设备隔离与限制。

- 网络隔离与限制。
- 用户、用户组隔离限制。

- ......

## 架构

![1624937894925-f437bd98-94e2-4334-9657-afa69bb52179](image/1624937894925-f437bd98-94e2-4334-9657-afa69bb52179.svg)

- Docker_Host

- - 安装Docker的主机

- Docker Daemon

- - 运行在Docker主机上的Docker后台进程

- Client

- - 操作Docker主机的客户端（命令行、UI等）

- Registry

- - 镜像仓库
  - Docker Hub

- Images

- - 镜像，带环境打包好的程序，可以直接启动运行

- Containers

- - 容器，由镜像启动起来正在运行中的程序

## 安装

### CentOS安装Docker

#### 移除以前的Docker安装包

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 配置yum源

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 安装Docker

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io


#以下是在安装k8s的时候使用
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```

#### 启动

```bash
systemctl enable docker --now
```

#### 配置加速

>这里额外添加了docker的生产环境核心配置cgroup

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```





### 其他系统

[其他系统参考此文档](https://docs.docker.com/engine/install/centos/)

