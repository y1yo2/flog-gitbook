---
description: 用于测试容器网络、镜像构建，或提供基础环境的docker镜像
---

# docker基础镜像

使用云环境的容器编排服务（Marathon、Kubernetes），镜像仓库（Container Repository）后，由于权限、网络环境等原因，导致无法命令行测试网络，镜像构建无法yum在线安装等。

需求：

1. 设置通用镜像，用于测试虚拟网络环境。
2. 设置基础镜像，作为业务应用镜像的基础镜像，且可复用。

### 任务一：操作系统镜像

1. 使用dockerHub的centos8镜像
2. 编写DockerFile构建通用镜像

{% embed url="https://hub.docker.com/\_/centos" caption="Docker Hub-Centos" %}

```bash
FROM centos:centos8 
MAINTAINER fanyy51 "fanyy51@chinaunicom.cn"

#set environment
ENV TIME_ZONE="Asia/Shanghai"

# set timezone
ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone

ENTRYPOINT ['tail', '-f', '/dev/null']
```

* Docker运行的监控进程，PID为1



* 前台进程，后台进程，设置监控进程防止容器退出



* 更优雅的解决方式









