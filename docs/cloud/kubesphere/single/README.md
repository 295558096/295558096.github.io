# Linux 单节点部署 KubeSphere

## 开通服务器

### 服务器配置

- 4c8g。
- centos7.9。
- 防火墙放行30000~32767。
- 指定hostname。

```bash
hostnamectl set-hostname node1
```

## 安装

### 准备KubeKey

```bash
export KKZONE=cn

curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.1 sh -

chmod +x kk
```

### 使用KubeKey引导安装集群

```bash
#可能需要下面命令
yum install -y conntrack

./kk create cluster --with-kubernetes v1.20.4 --with-kubesphere v3.1.1
```

## 安装后开启功能

![open](image/open.png)

