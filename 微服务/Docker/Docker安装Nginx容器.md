# Docker 安装 Nginx 容器

[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)如果想安装软件 , 必须先到 [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020) 镜像仓库下载镜像。

[Docker官方镜像](https://hub.docker.com/)

## 1、寻找[Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020)镜像 

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189052726-3ca6b16e-0536-4be1-aede-1272dc26d249.png)

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189057170-116ed974-9809-4f39-95f3-1b4c2e397446.png)

## 2、下载Nginx镜像

| **命令**              | **描述**                                                     |
| --------------------- | ------------------------------------------------------------ |
| docker pull nginx     | 下载最新版Nginx镜像 (其实此命令就等同于 : docker pull nginx:latest ) |
| docker pull nginx:xxx | 下载指定版本的Nginx镜像 (xxx指具体版本号)                    |

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189065253-04d6e215-b70b-4330-b7e0-7b1131f8d8a9.png)

检查当前所有[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)下载的镜像

```bash
docker images
```

 3、创建Nginx配置文件 

启动前需要先创建Nginx外部挂载的配置文件（ /home/nginx/conf/nginx.conf）

之所以要先创建 , 是因为Nginx本身容器只存在/etc/nginx 目录 , 本身就不创建 nginx.conf 文件

当服务器和容器都不存在 nginx.conf 文件时, 执行启动命令的时候 docker会将nginx.conf 作为目录创建 , 这并不是我们想要的结果 。

```bash
# 创建挂载目录
mkdir -p /home/nginx/conf
mkdir -p /home/nginx/log
mkdir -p /home/nginx/html

# 创建临时容器,将容器中的nginx.conf文件和conf.d文件夹复制到宿主机

# 生成容器
docker run --name nginx -p 9001:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/nginx/

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
-p 9002:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189187948-c0e4a5c6-423a-46c6-8c4d-9567d1a6eecd.png)

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189192913-df64a121-3109-4183-8b53-1fbbd2def874.png)

单行模式

```bash
docker run -p 9002:80 --name nginx -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx/conf/conf.d:/etc/nginx/conf.d -v /home/nginx/log:/var/log/nginx -v /home/nginx/html:/usr/share/nginx/html -d nginx:latest
```

## 3、结果检测

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189210919-88dd9b6a-99e3-43ed-8cd0-82edf678c9d3.png)

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189215701-5fbc1ef6-4ded-4c25-a814-5592374d0819.png)

## 4、修改内容进行展示

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189224241-9a59566f-392b-41c3-838c-7b61d5b45499.png)

\# 重启容器

```bash
docker restart nginx
```

![img](Docker%E5%AE%89%E8%A3%85Nginx%E5%AE%B9%E5%99%A8.assets/1700189243061-3100b24c-65d4-4995-95f1-d40619025e00.png)