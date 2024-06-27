## Docker安装Tomcat

-  tomcat 默认安装目录在/usr/local/tomcat下 
-  高版本tomcat 默认文件地址在webapps.dist 下，需要挂载，需要将webapps.dist文件下的目录copy到webappps下 

### Tomcat DockerHub官方地址

下载最新版Tomcat

```bash
[root@localhost /]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
......
4b7c9018ca5f: Pull complete 
Digest: sha256:46456ccf216f2fde198844d5c7d511f60e3ffc2ea3828e0cce9a2eed566e48e2
Status: Downloaded newer image for tomcat:latest
docker.io/library/tomcat:latest
```

### 查看并启动容器

```bash
[root@localhost /]# docker run -it -d -p 8080:8080 --name tomcat tomcat 
decea5a47b7f771aa2357468416897080b470099378c5300a79fbd5eddbacfce
```

### 命令示意

| 命令       | 示意                                                       |
| ---------- | ---------------------------------------------------------- |
| docker run | 启动容器                                                   |
| -it        | 以交互式启动                                               |
| -d         | 后台运行                                                   |
| -p         | 指定容器暴露出来的端口号:镜像容器内部的端口号              |
| -P         | 随机指定端口号                                             |
| -v         | 数据卷挂载 /宿主机目录地址:/容器内部地址（后面可以跟多个） |
| –name      | 指定启动名称                                               |

### 访问地址发现404

![img](Docker%E5%AE%89%E8%A3%85Tomcat.assets/202305272200518.png)

### 高版本Tomcat 默认文件地址是在webapps.dist里面，需要将webapps.dist文件下的内容复制到webapps下

进入容器内部，

```bash
#进入 容器内部
[root@localhost ~]# docker exec -it decea5a47b7f /bin/bash
# 查看容器Tomcat列表
root@decea5a47b7f:/usr/local/tomcat# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE  NOTICE  README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  native-jni-lib  temp  webapps  webapps.dist  work
#此时webapps 文件夹里面的内容是空的，复制 webapps.dist 所有文件到 webapps/
root@decea5a47b7f:/usr/local/tomcat# cp -r webapps.dist/* webapps/
root@decea5a47b7f:/usr/local/tomcat# cd webapps
#再次查看
root@decea5a47b7f:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager
root@decea5a47b7f:/usr/local/tomcat/webapps# 
#退出重启容器
root@decea5a47b7f:/usr/local/tomcat/webapps# exit
exit
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
decea5a47b7f        tomcat              "catalina.sh run"        16 minutes ago      Up 16 minutes       0.0.0.0:8080->8080/tcp              tomcat
3b013a36bbb6        mysql:5.7           "docker-entrypoint.s…"   58 minutes ago      Up 58 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
[root@localhost ~]# docker stop decea5a47b7f
decea5a47b7f
[root@localhost ~]# docker start decea5a47b7f
decea5a47b7f
```

## Tomcat 挂在数据卷

```bash
# 以挂载数据卷方式启动Tomcat
[root@localhost /]# docker run -it -d -p 8080:8080 -v /data/docker/tomcat/logs:/usr/local/tomcat/logs -v /data/docker/tomcat/webapps:/usr/local/tomcat/webapps --name tomcat tomcat
27c6b82b56e8793f1ceb50de923cdcfb20b6626df8b600d939aee74516757618

# 将日志和webapps 分别进行挂载
```

**已经挂载的目录无法二次挂载**

### 查看宿主机的文件目录和挂载情况

```powershell
[root@localhost webapps]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
27c6b82b56e8        tomcat              "catalina.sh run"        4 minutes ago       Up 4 minutes        0.0.0.0:8080->8080/tcp              tomcat
3b013a36bbb6        mysql:5.7           "docker-entrypoint.s…"   About an hour ago   Up About an hour    0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
[root@localhost webapps]# docker inspect 27c6b82b56e8
......
   "Mounts": [
       {
           "Type": "bind",
           "Source": "/data/docker/tomcat/logs",
           "Destination": "/usr/local/tomcat/logs",
           "Mode": "",
           "RW": true,
           "Propagation": "rprivate"
       },
       {
           "Type": "bind",
           "Source": "/data/docker/tomcat/webapps",
           "Destination": "/usr/local/tomcat/webapps",
           "Mode": "",
           "RW": true,
           "Propagation": "rprivate"
       }
   ],
#查看宿主机文件目录
[root@localhost webapps]# pwd
/data/docker/tomcat/webapps
[root@localhost webapps]# ll
total 4
drwxr-xr-x. 16 root root 4096 Jul 18 20:22 docs
drwxr-xr-x.  6 root root   83 Jul 18 20:22 examples
drwxr-xr-x.  5 root root   87 Jul 18 20:22 host-manager
drwxr-xr-x.  5 root root  103 Jul 18 20:22 manager
drwxr-xr-x.  3 root root  283 Jul 18 20:22 ROOT
# 文件映射成功，且创建文件后能自动同步
```