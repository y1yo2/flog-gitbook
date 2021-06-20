---
description: 弄一个启动jar包的脚本
---

# 项目启动脚本

\#!/bin/bash，解释该脚本的解释器。不同系统创建的shell默认解释器。

bash功能最全效率低，sh轻量但不100%兼容。

通过拼接JAVA\_OPT、JVM\_XMS、JVM\_XMX、JVM\_XMN、JVM\_MS、JVM\_MMS，字符串变量，设置启动参数。

$JAVA\_OPT 或 ${JAVA\_OPT}，引用字符串变量。

 `ps -ef`命令是Process Status，-e\|-A：显示所有进程，-f：全格式显示进程。

管道`|`将上条命令输出作为下条命令的输入；`grep`命令过滤输入内容（文件），得到包含指定字符的字符串，加`-v`参数表示不包含；

命令`awk -F '分隔符默认为空格' '{print $1}' file.txt`分割过滤输入内容（字符串）；

```bash
pidstr=`ps -ef|grep ***.jar|grep -v 'grep'|awk '{print $2}'`
```

列出所有进程，过滤只包含特定jar的进程，过滤不包含“grep”字符串的进程，分割内容，输出第二个字符串，得到pid。

```bash
java $JAVA_OPT -jar ***.jar >/dev/null 2>&1 &
```







