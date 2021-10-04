---
description: idea的File Header；Git的name和email等；
---

# 工程作者信息

## idea

### 一、文件Header的 template

配置路径：File &gt; Settings &gt; Editor &gt; File and Code Templates &gt; Includes

```java
/**
 * @author username
 * @date ${DATE} ${TIME}
 */
```

## git

### 一、提交代码的用户名和邮箱

修改全局用户名、邮箱的方式有：

1. 全局配置文件 .gitconfig（C:\Users\username）
2. 命令行如下：

```bash
git config --global user.name username
git config --global user.email email
```

查看并设置当前项目的用户名、邮箱：

```bash
# 查看
git config user.name
git config user.email

# 设置
git config user.name username
git config user.email email
```

### 二、git回滚代码且删除远程记录

1、查看commitID：`git log` 

2、回滚本地文件：`git reset --hard {commitID}` 

3、将回滚结果，推送远程：`git push -f`   









