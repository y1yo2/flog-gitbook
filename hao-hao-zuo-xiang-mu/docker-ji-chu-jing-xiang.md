---
description: 用于测试容器网络、镜像构建，或提供基础环境的docker镜像
---

# docker基础镜像

使用云环境的容器编排服务（Marathon、Kubernetes），镜像仓库（Container Repository）后，由于权限、网络环境等原因，导致无法命令行测试网络，镜像构建无法yum在线安装等。

需求：

1. 设置通用镜像，用于测试虚拟网络环境。
2. 设置基础镜像，作为业务应用镜像的基础镜像，且可复用。

## 任务一：操作系统镜像

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

* CMD 与 ENTRYPOINT

### 关于CMD：

1. Dockerfile存在多个CMD，只有最后一个CMD（命令行CMD覆盖镜像CMD）生效。
2. CMD的主要目的是为正在启动的容器提供默认值。
3. 如果存在ENTRYPOINT，则CMD忽略可执行部分，只为ENTRYPOINT提供默认值。
4. Shell格式（shell文件中的命令），exec-可执行模式无法执行。需要使用`CMD["sh", "-c", "echo $test"]`，通过sh执行命令。

> 用法：
>
> CMD \["可执行文件", "param"\]   （推荐用法）
>
> CMD \["param"\]   （作为 ENTRYPOINT 的默认参数）
>
> CMD 命令 param

### 关于ENTRYPOINT：

1. 如果希望容器每次运行的可执行文件都相同，应使用 ENTRYPOINT 防止覆盖。
2. `docker run <image> param`参数将附加在ENTRYPOINT后。CMD同理。
3. shell格式的ENTRYPOINT，会作为`/bin/sh -c`子命令启动。因为运行状态/信息无法监测，所以该命令无法作为容器的`PID 1`进程。
4. 推荐使用ENTRYPOINT定义默认命令和参数，使用CMD设置临时参数。

> 用法：
>
> ENTRYPOINT \["可执行文件", "param"\]（推荐用法）
>
> ENTRYPOINT 命令 param
>
>
>
> 疑问：
>
> /bin/sh -c，为什么无法传递状态？
>
>
>
> 基础镜像的CMD，能否被其他镜像执行？
>
> 其他镜像设置ENTRYPOINT，基础镜像的CMD将被置为空。

* Docker运行的监控进程，PID为1



* 前台进程，后台进程，设置监控进程防止容器退出

docker容器生命周期绑定监控进程，若监控进程运行结束，则容器自动停止。

因此通用镜像会使用 tail -f /dev/null 等，开启一个前台的监控进程，防止容器停止。

* 更优雅的解决方式

使用docker的 interactive 和 tty 参数，将 sh/bash（linux系统自带）作为前台的进程。

> 注意使用的基础镜像，是否自带ENTRYPOINT。docker run传入的参数将无效，需使用 --entrypoint 覆盖。
>
> 若出现权限不足，使用 --privileges = true。



## 任务二：可复用基础镜像









