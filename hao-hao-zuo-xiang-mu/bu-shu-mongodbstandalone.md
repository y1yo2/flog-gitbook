---
description: 2021/09/01
---

# 部署MongoDB-standalone

快速部署一个单机模式的MongoDB，供同事测试使用。

### 1.安装&#x20;

1.1 安装依赖包： `sudo yum install libcurl openssl`&#x20;

1.2 安装mongodb：\
&#x20;[https://www.mongodb.com/download-center#community](https://www.mongodb.com/download-center#community)&#x20;

使用包管理器 yum 或 apt \
或者下载 TGZ or ZIP file \
`tar -zxvf ***.zip` \
`mv mongodb /usr/local/mongodb`

### 2.配置文件&#x20;

mongodb文档：[https://docs.mongodb.com/v4.2/reference/configuration-options/index.html](https://docs.mongodb.com/v4.2/reference/configuration-options/index.html) \
zip文件安装需手动增加配置文件 /etc/mongod.conf

### 3.配置可执行文件

```bash
sudo vim /etc/profile 
export PATH=/usr/local/mongodb/bin:$PATH 
sudo source /etc/profile
```

### 4.创建数据存储目录、日志存储目录（与配置文件对应）&#x20;

`sudo mkdir -p /var/lib/mongo` \
`sudo mkdir -p /var/lib/mongodb`

### 5.启动&#x20;

`mongod --config /etc/mongod.conf`

### 6.创建mongo超级管理员账号&#x20;

```bash
sudo ./mongo
show dbs 
db（MongoDB 中默认的数据库为 test） 
use admin 
show users 
db.createUser({user:"root",pwd:"123456",roles:[{ role:"root", db:"admin" } ] }) 
quit()
```

### 7.设置mongodb开启认证鉴权&#x20;

```bash
vim /etc/mongod.conf 
security: 
  authorization: enabled

sudo systemctl restart mongod
```

### 8.创建自己的db和账号&#x20;

`sudo ./mongo -u root`\
`use mydb db.createUser({user:"myaccount",pwd:"123456",roles:[{role:"dbOwner", db:"mydb"}]})`

### 9.防火墙放通&#x20;

```bash
sudo firewall-cmd --get-active-zones 
sudo firewall-cmd --zone=public --list-ports 
sudo firewall-cmd --zone=public --add-port=27017/tcp --permanent # 永久生效
sudo systemctl reload firewalld 
sudo systemctl restart firewalld
```

### 10.客户端连接MongoDB

客户端推荐 Robo 3T (Robomong) 推荐 官网下载地址：[https://robomongo.org/download](https://robomongo.org/download)

MongoDB Compass（MongoDB子产品） 官网下载地址：[https://www.mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass) 官网教程：[https://docs.mongodb.com/manual/reference/connection-string/](https://docs.mongodb.com/manual/reference/connection-string/)
