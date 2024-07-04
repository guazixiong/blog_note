# Centos 7 安装开发环境

## Java8

### 卸载系统自带jdk

```bash
# 查看JDK安装版本
[root@pbad ~]#java -version
java version *1.7.0_51*
OpenJDK Runtime Environment ( rhel-2.4.5.5.el7-x86_64 u51-b31)
OpenJDK 64-Bit Server VM (build 24.51-b03, mixed mode)

# 查找OpenJDK安装包
[root@pbad ~]# rpm -qa | grep openjdk
java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64
java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64

# 卸载OpenJDK安装包(有多少卸载多少)
[root@pbad ~]# yum -y remove java-1.7.0-openjdk-headless.x86_64
[root@pbad ~]# yum -y remove java-1.7.0-openjdk.x86_64

# 查看是否卸载成功
[root@pbad ~]# java -version
```

### 安装jdk

```bash
# wget方式获取
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"

# 解压jdk8
tar -zxvf jdk-8u141-linux-x64.tar.gz

# 移动目录
mv jdk1.8.0_141 /usr/local/jdk8
```

### 配置环境

修改 `profile` 配置文件：

```bash
# 修改 `profile` 配置文件
vim /etc/profile

export JAVA_HOME=/usr/local/jdk8 # 刚刚jdk移动到的目录
export MAVEN_HOME=/usr/local/apache-maven-3.9.6 # 刚刚maven移动到的目录
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$MAVEN_HOME/bin:$PATH

# 使配置文件生效
source /etc/profile

# 查看mvn版本
java -version

java version "1.8.0_141"
Java(TM) SE Runtime Environment (build 1.8.0_141-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.141-b15, mixed mode)
```

## Maven

### 下载安装

手动下载maven安装包. Maven – Welcome to Apache Maven: https://maven.apache.org/

利用sftp文具将安装包上传至服务器

```bash
# 解压
tar -zxvf apache-maven-3.9.6

# 移动目录
mv apache-maven-3.9.6 /usr/local/apache-maven-3.9.6
```

### 配置环境

修改 `profile` 配置文件：

```bash
# 修改 `profile` 配置文件
vim /etc/profile

export JAVA_HOME=/usr/local/jdk8 # 刚刚jdk移动到的目录
export MAVEN_HOME=/usr/local/apache-maven-3.9.6 # 刚刚maven移动到的目录
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$MAVEN_HOME/bin:$PATH

# 使配置文件生效
source /etc/profile

# 查看mvn版本
mvn -v

Apache Maven 3.9.6 (bc0240f3c744dd6b6ec2920b3cd08dcc295161ae)
Maven home: /usr/local/apache-maven-3.9.6
Java version: 1.8.0_141, vendor: Oracle Corporation, runtime: /usr/local/jdk8/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.el7.x86_64", arch: "amd64", family: "unix"

export JAVA_HOME=/usr/local/jdk8
export MAVEN_HOME=/usr/local/apache-maven-3.9.8
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$MAVEN_HOME/bin:$PATH
```

## Git

### 下载并安装

```bash
# 下载 
[root@pbad ~]# yum install git

# 查看git版本
[root@pbad ~]# git --version
git version 1.8.3.1
```

### 配置

```bash
# 配置基本信息
[root@localhost ~]# git config --global user.name "pbad"
[root@localhost ~]# git config --global user.email "xxx@163.com"

# 查看配置
[root@localhost ~]# git config --list
user.name=pbad
user.email=xxx@163.com
```

### 配置ssh key

工作台 - Gitee.com: https://gitee.com/

GitHub: https://github.com/

## NVM

### 下载安装

官网：https://github.com/nvm-sh/nvm

方式一：命令行工具输入，通过curl工具下载

```bash
# 未安装curl需要先安装
sudo yum install curl -y

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

方式二：命令行工具输入，通过wget工具下载

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

方式一、方式二可能会出现443下载失败,原因是被墙了

```bash
Failed connect to raw.githubusercontent.com:443; Connection refused
```

解决方式如下:

1. 服务器翻墙
2. 修改host文件,多试几次 

> 推荐ip查询网址 https://www.ipaddress.com/,查询`github.com` 和 `raw.githubusercontent.com`的具体id并修改host文件即可。

![image-20231202230624258](Centos%207%20%E5%AE%89%E8%A3%85%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83.assets/image-20231202230624258.png)

 修改host文件

```bash
sudo vim /etc/hosts

140.82.112.3 github.com
185.199.108.133 raw.githubusercontent.com
```

方式三: 下载安装包

nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions --- nvm-sh/nvm：节点版本管理器 - 符合 POSIX 的 bash 脚本，用于管理多个活动节点 .js 版本: https://github.com/nvm-sh/nvm

```bash
# 解压
unzip nvm-0.39.3.zip
cd nvm-0.39.3/
sh install.sh
```

### 修改配置文件

```bash
# 镜像修改为国内镜像
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node/
export NVM_IOJS_ORG_MIRROR=http://npm.taobao.org/mirrors/iojs

# 查看远程库可以安装的 node 版本
nvm ls-remote

# npm 镜像更改为国内镜像
npm config set sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
npm config set phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs/
npm config set electron_mirror=https://npm.taobao.org/mirrors/electron/
npm config set registry=https://registry.npm.taobao.org

# 查看镜像配置
npm config ls

[root@192 pbad]# npm config ls
; cli configs
metrics-registry = "https://registry.npm.taobao.org/"
scope = ""
user-agent = "npm/6.13.4 node/v12.16.0 linux x64"

; userconfig /root/.npmrc
electron_mirror = "https://npm.taobao.org/mirrors/electron/"
phantomjs_cdnurl = "https://npm.taobao.org/mirrors/phantomjs/"
registry = "https://registry.npm.taobao.org/"
sass_binary_site = "https://npm.taobao.org/mirrors/node-sass/"

; node bin location = /root/.nvm/versions/node/v12.16.0/bin/node
; cwd = /home/pbad
; HOME = /root
; "npm config ls -l" to show all defaults.
```

### 安装指定node

```bash
nvm install v14.16
```

### 常用命令

```bash
# 查看已经安装的node版本
nvm ls
# 使用最新版本
nvm use latest
# 使用长期支持版本
nvm use lts
# 使用你已安装的任何其他版本
nvm use 14.20.0

# 查看nvm版本
nvm -v
# 查看node版本
node -v
# 查看npm版本
npm -v
```

## Docker

```bash
# 1.卸载旧版本
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2.需要的安装包
sudo yum install -y yum-utils

# 3.设置镜像仓库(默认是国外地址)
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 3.设置国内镜像地址, 如阿里云镜像地址
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 4.更新yum 索引 
yum makecache fast
    # 如果报错yum makecache: error: argument timer: invalid choice: 'fast' (choose from 'timer')
    yum makecache    # 执行

# 5.安装docker相关内容 docker-ce 社区版 ee 企业版
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 6.启动docker
sudo systemctl start docker

# 7.查看docker 版本
docker version

# 8.通过运行 hello-world 映像来验证 Docker Engine 安装是否正确
sudo docker run hello-world

# 9. 配置镜像加速

# 创建docker配置文件
sudo mkdir -p /etc/docker
# 追加配置文件
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://nf9shctl.mirror.aliyuncs.com"]
}
EOF

# 重新加载配置文件
sudo systemctl daemon-reload

# 重启docker容器
sudo systemctl restart docker
```

## Docker-Nginx

```bash
# docker安装最新版nginx
docker pull nginx

# 创建挂载目录
mkdir -p /usr/local/nginx/conf
mkdir -p /usr/local/nginx/log
mkdir -p /usr/local/nginx/html

# 创建临时容器,将容器中的nginx.conf文件和conf.d文件夹复制到宿主机

# 生成容器
docker run --name nginx -p 9001:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /usr/local/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /usr/local/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /usr/local/nginx/

# 移除临时容器
# 直接执行docker rm nginx或者以容器id方式关闭容器
# 找到nginx对应的容器id
docker ps -a
# 关闭该容器
docker stop nginx
# 删除该容器
docker rm nginx
	# 删除正在运行的nginx容器
	docker rm -f nginx

# 创建最终的执行容器
docker run \
-p 9001:80 \
--name nginx \
-v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /usr/local/nginx/log:/var/log/nginx \
-v /usr/local/nginx/html:/usr/share/nginx/html \
-d nginx:latest

# 验证是否安装成功,出现以下内容说明安装成功
          

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

## Docker-Redis

```bash
# 检索redis镜像
docker search Redis

# 从docker hub上(阿里云加速器)拉取redis镜像
docker pull redis

# 查看本地镜像
docker images

# 新增本地文件夹,挂载redis配置文件
mkdir -p /usr/local/redis
mkdir -p /usr/local/redis/data
mkdir -p /usr/local/redis/conf

# 编辑本地文件下的redis.conf文件
cd /usr/local/redis/conf
vim redis.conf

# 追加参数
daemonize no
appendonly yes
tcp-keepalive 300

# 创建容器并启动(对外端口号6379,挂载数据在mkdir /usr/local/redis,配置redis.conf,启动redis-server,开启redis持久化机制)
docker run --name redis \
-p 6379:6379 \
-v /usrl/local/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /usrl/local/redis:/data \
-d redis redis-server /etc/redis/redis.conf --appendonly yes

# 客户端连接测试
 docker exec -it redis redis-cli
 
ping

set k1 v1

get k1

exit
```

## Docker-Mysql

```bash
# 创建mysql挂载目录
sudo mkdir -p /usr/local/mysql
sudo mkdir -p /usr/local/mysql/conf
sudo mkdir -p /usr/local/mysql/data

# 创建自定义配置文件myconf.cnf
cd /usr/local/mysql
chmod 777 conf
cd conf
touch myconf.cnf 

# docker 中下载 mysql
docker pull mysql:5.7

# 创建容器 映射3306目录,挂载自定义配置文件和数据,初始化root用户密码
docker run --name mysql -p 33060:3306 \
-v /usr/local/mysql/conf:/etc/mysql/conf.d \
-v /usr/local/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

# 进入容器
docker exec -it mysql bash

# 登录mysql
mysql -u root -p
# 修改加密规则（可以直接复制）
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
# 修改 MySQL 中 root 用户在 localhost 上的密码为 gpgg6bp3quhqhadbnhcw。
ALTER USER 'root'@'localhost' IDENTIFIED BY 'gpgg6bp3quhqhadbnhcw';

# 添加远程登录用户jiyuan,密码jiyuan4zhong
create user 'jiyuan'@'%' identified by 'jiyuan4zhong';
# 授予该用户对所有数据库的所有权限
GRANT ALL PRIVILEGES ON *.* TO 'jiyuan'@'%';

# 刷新权限（可以直接复制）
FLUSH PRIVILEGES;
```

## Docker-tomcat

### 简易安装

### 挂载目录

```bash
# 创建tomcat挂载目录
sudo mkdir -p /usr/local/tomcat
sudo mkdir -p /usr/local/tomcat/logs
sudo mkdir -p /usr/local/tomcat/webapps

# 下载tomcat
docker pull tomcat

# 创建容器,映射端口8080,挂载logs目录和webapps目录
docker run -it -d -p 8080:8080 \
-v /data/docker/tomcat/logs:/usr/local/tomcat/logs \
-v /data/docker/tomcat/webapps:/usr/local/tomcat/webapps \
--name tomcat tomcat

# 测试是否安装完成
curl http://localhost:8080

# 出现404,进入容器(高版本Tomcat 默认文件地址是在webapps.dist里面,访问8080会404)
docker exec -it decea5a47b7f /bin/bash

# webapps 文件夹里面的内容是空的，复制 webapps.dist 所有文件到 webapps/
cp -r webapps.dist/* webapps/

# 退出,重启容器
exit
docker restart tomcat

# 测试是否安装完成
curl http://localhost:8080
```



