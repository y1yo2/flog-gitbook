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

{% embed url="https://hub.docker.com/_/centos" %}
Docker Hub-Centos
{% endembed %}

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
> CMD \["可执行文件", "param"]   （推荐用法）
>
> CMD \["param"]   （作为 ENTRYPOINT 的默认参数）
>
> CMD 命令 param

### 关于ENTRYPOINT：

1. 如果希望容器每次运行的可执行文件都相同，应使用 ENTRYPOINT 防止覆盖。
2. `docker run <image> param`参数将附加在ENTRYPOINT后。CMD同理。
3. shell格式的ENTRYPOINT，会作为`/bin/sh -c`子命令启动。因为运行状态/信息无法监测，所以该命令无法作为容器的`PID 1`进程。
4. 推荐使用ENTRYPOINT定义默认命令和参数，使用CMD设置临时参数。

> 用法：
>
> ENTRYPOINT \["可执行文件", "param"]（推荐用法）
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



### 生命周期与前台进程

* 前台进程，后台进程，设置监控进程防止容器退出

docker容器生命周期绑定监控进程，若监控进程运行结束，则容器自动停止。

因此通用镜像会使用 tail -f /dev/null 等，开启一个前台的监控进程，防止容器停止。

* 更优雅的解决方式

使用docker的 interactive 和 tty 参数，将 sh/bash（linux系统自带）作为前台的进程。

> 注意使用的基础镜像，是否自带ENTRYPOINT。docker run传入的参数将无效，需使用 --entrypoint 覆盖。
>
> 若出现权限不足，使用 --privileges = true。



## 任务二：可复用基础镜像

### 关于ENV

Dockerfile设置容器的环境变量。

* 定义ENV时，可使用已定义的环境变量。
* 默认环境变量有$HOME, $PATH, $HOSTNAME。
* 镜像是层次文件系统，一条指令一个镜像层。因此ENV定义的变量，在后续层次才能使用。同一层ENV无法相互引用。

```bash
ENV a=a1
ENV a=a2 b=$a
ENV c=$a

env #查看容器的环境变量。b=a1, c=a2 
```

### 关于环境变量

* 设置环境变量

`/etc/profile` 设置变量后，对linux下所有用户有效。

`source /etc/profile` 让修改马上生效。

`~/.bash_profile` 设置变量，仅对linux下当前用户有效。

`source ~/.bash_profile` 让修改马上生效

`export PATH=/home/me/apps:$PATH` 设置变量，仅对当前shell(sh/bash)有效。直接生效，关闭shell后失效。

* 查看环境变量

&#x20;`echo $SHELL` 显示一个环境变量

&#x20;`env`  显示所有环境变量

&#x20;`unset $TEST`  删除一个环节变量

&#x20;`readonly $TEST`  设置变量为只读，不可修改删除

### 关于执行shell脚本（exec模式）

&#x20;`./xxx.sh` 使用shell脚本内指定的sh/bash作为subshell（子shell）执行，但shell脚本需要**执行权限**。 &#x20;

&#x20;`sh/bash xxx.sh` 使用sh/bash作为subshell（子shell）执行，shell脚本无需执行权限。 &#x20;

&#x20;`source xxx.sh`  或 `. xxx.sh` 使用当前shell执行shell脚本，脚本无需执行权限。 &#x20;

### 关于启动进程参数

执行shell命令时，默认打开3个文件。

|         **类型**        | **文件描述符** |   **默认情况**  |   **对应文件句柄位置**  |
| :-------------------: | :-------: | :---------: | :-------------: |
|  标准输入（standard input） |     0     |   从键盘获得输入   | /proc/slef/fd/0 |
| 标准输出（standard output） |     1     | 输出到屏幕（即控制台） | /proc/slef/fd/1 |
|   错误输出（error output）  |     2     | 输出到屏幕（即控制台） | /proc/slef/fd/2 |

因此命令执行时，默认从键盘获取输入（使用0），结果输出到屏幕（使用1）。

* 输出输入重定向

| **命令**      | **作用**       |
| ----------- | ------------ |
| 1>filename  | 标准输出重定向到file |
| >filename   | 同上           |
| 2>filename  | 错误输出重定向到file |
| 0\<filename | file作为标准输入   |
| \<filename  | 同上           |

* 常用的重定向

&#x20;`>/dev/null` ，标准输出指向/dev/null（linux的空设备文件，往该文件写入内容会丢失）。

&#x20;`2>&1` ，使用&将两个输出绑定，错误输出指向标准输出。

&#x20;`>/dev/null 2>&1` ，先将标准输出指向/dev/null，再将错误输出指向标准输出即/dev/null。

### 关于前台进程、后台进程

前台进程：进程执行情况输出shell终端。终端关闭会导致进程终止。

* 后台形式启动进程

命令末尾加 & 符号，后台运行进程。但终端关闭，进程会终止。

nohup 加上后，用户退出或终端关闭都不影响当前后台进程。

* 查看进程

&#x20;`jobs -l` ，查看后台进程。

&#x20;`ps -aux`，查看所有进程。

&#x20;`kill %jobid` ， `kill pid` ，终止进程。

* 运行中进程放入后台、前台

运行中的前台进程，ctrl + z 放入后台并暂停。

&#x20;`bg %jobid` 将暂停的后台进程运行。

&#x20;`fg %jobid` 将后台进程放上前台。



