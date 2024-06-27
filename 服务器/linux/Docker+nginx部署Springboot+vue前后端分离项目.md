# Docker+nginx部署Springboot+vue前后端分离项目

> 原文链接: 手把手教你Docker+nginx部署Springboot+vue前后端分离项目 - Java问答社: https://www.zhuawaba.com/post/84
>
> 项目代码:   一个Vue+springboot前后端分离的微社区项目: 一个Vue+springboot前后端分离的微社区项目: https://gitee.com/Talon_index/newRetwis

同步演示怎么部署到win环境和linux（centos7）系统中，前端服务器采用nginx部署，并使用docker统一管理前后端服务器。

所以我们会用到：

- nginx
- docker-compose

## Windows部署

### 工具

+ 安装windwos版本nginx
+ 安装数据库连接工具,如Navicat

### 前端操作

先来配置一下后端的调用路径，因为现在部署在本地localhost，所以在axios.js中，我们配置好链接，因为等下后端部署也是本机，所以我这里直接这样配置就好了，如下：

`src\axios.js`,配置的就是前端访问后端接口的服务。然后前端部署还需要考虑一个问题：打包之后项目资源引用路径，比如我们打包后链接是否需要带项目名称等。按照Vue官方的部署说明，我们添加一个**vue.config.js**文件，

```js
axios.defaults.baseURL = "http://localhost:8081"
```

`vueblog-vue/vue.config.js`

```js
module.exports = {
publicPath: '/'
}
```

当然了，publicPath默认其实是空的，也就是`publicPath: ‘’`

执行打包命令,会在根目录下产生`dist`目录,这个就是打包之后的文件夹，里面有个`index.html`，但是我们点击直接打开是看不到任何内容的，这时候，我们需要将`dist`部署到`nginx`中

```bash
npm run build
```

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/e6f843e72b754170979ff89d30fa4ddc.png)

清空nginx目录下的`nginx/html`目录下自带的`50x.html`和`index.html`,将我们打包好的`dist`文件夹下的内容复制粘贴

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/b9c6f2e4f249418aaac8fb9b1cf76db9.png)



启动nginx,浏览器访问`http://localhost`,出现前端页面(非welcome to nginx)

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/f62108a4241e4b22a8efb9f9db89e963.png)

> 我们点击任意一个链接或者按钮或者刷新界面，这时候出现了404
>
> 刷新之后nginx就找不到路由了,vue项目的入口是index.html文件，nginx路由的时候都必须要先经过这个文件，所以我们得给nginx定义一下规则，让它匹配不到资源路径的时候，先去读取index.html，然后再路由。所以我们配置一下nginx.conf文件。具体操作就是找到**location /**,添加上一行代码**try_files $uri $uri/ /index.html last**;如下：
>
> nginx-1.18.0/conf/nginx.conf
>
> ```bash
> location / {
>   root   html;
>   try_files $uri $uri/ /index.html last;
>   index  index.html index.htm;
> }
> ```
>
> 1. `$uri`: 这个变量代表当前请求的 URI。Nginx 首先会尝试使用当前请求的 URI 查找对应的文件。
> 2. `$uri/`: 如果第一步未找到对应的文件，Nginx 将会尝试使用当前请求的 URI 作为目录名，查找对应目录下的默认文件。比如，如果用户请求的是 `/example/`，那么 Nginx 将会尝试查找 `/example/index.html`。
> 3. `/index.html`: 如果前两步都未找到对应的文件或目录，默认返回 `/index.html` 页面。
> 4. `last`: 这个参数告诉 Nginx 在尝试完当前 `location` 块内的所有指令后，如果未能成功处理请求，则将控制权传递给下一个 `location` 块。
>
> 综合起来，`try_files $uri $uri/ /index.html last;` 这个指令的作用是：尝试根据请求的 URI 查找对应的文件，如果找不到，则尝试查找对应目录下的默认文件，最终如果仍然找不到，则返回 `/index.html` 页面。
>
> 温馨提示: 修改nginx.conf后,请重启nginx服务 (windows环境nginx的重启可以通过通过打开任务管理器直接干掉nginx进程，然后再重新双击)



### 后端操作

pom.xml文件放开`spring-boot-maven-plugin`

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

执行打包命令:

```bash
# 跳过测试打包
mvn clean package -Dmaven.test.skip=true
```

得到`target`目录下的`vueblog-0.0.1-SNAPSHOT.jar`，然后再执行命令,加载`application-default.yml`配置信息

```bash
java -jar vueblog-0.0.1-SNAPSHOT.jar --spring.profiles.active=default
```

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/7349025903a2439ba475a758f91935c8.png)

通过`http://localhost`访问我们的前端项目和后端接口联动效果.

## Linux部署

### 工具

安装`docker`

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

安装`docker-compose`

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 赋权
sudo chmod +x /usr/local/bin/docker-compose
# 检查是否安装成功
docker-compose --version
```

### 前端操作

跟windows部署前端一样,执行`npm run build`打包,获取`dist`目录

### 后端操作

#### Dockerfile

在根目录创建`Dockerfile`文件

```bash
FROM java:8
EXPOSE 8080
ADD vueblog-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
# 追加线上环境配置参数
# ENTRYPOINT ["java", "-jar", "/app.jar", "--spring.profiles.active=pro"]
# 使用默认环境配置参数
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

> 依赖于java8,对外暴露端口为8081,复制当前目录下的`vueblog-0.0.1-SNAPSHOT.jar`到docker容器中并命名为`app.jar`,最后是执行命令**java -jar /app.jar —spring.profiles.active=pro**，使用的是我们另外编写的一个线上环境配置。

#### docker-compose

在根目录创建`docker-compose.yml`文件

```yaml
version: "3"
services:
  nginx: # 服务名称，用户自定义
    image: nginx:latest  # 镜像版本
    ports:
      - 80:80  # 暴露端口
    volumes: # 挂载
      - /root/nginx/html:/usr/share/nginx/html
      - /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
      - /root/nginx/log:/var/log/nginx
    privileged: true # 这个必须要，解决nginx的文件调用的权限问题
   mysql:
     image: mysql:5.7.27
     ports:
      - 3306:3306
     environment: # 指定用户root的密码
        - MYSQL_ROOT_PASSWORD=admin
   redis:
     image: redis:latest
   vueblog:
     image: vueblog:latest
     build: . # 表示以当前目录下的Dockerfile开始构建镜像
     ports:
      - 8081:8081
     depends_on: # 依赖与mysql、redis，其实可以不填，默认已经表示可以
       - mysql
       - redis
```

> 1. `version: "3"`：指定了 Docker Compose 文件的版本。这个配置文件符合 Docker Compose 版本 3 的语法规范。
> 2. `services:`：这是一个服务部分的开始标志，下面列出了各个服务的配置信息。
> 3. `nginx:`：这是一个服务名称，用户自定义的，代表一个运行着 Nginx 的容器。
>    - `image: nginx:latest`：使用的镜像是最新版本的 Nginx。Docker 将会拉取这个镜像并在容器中运行 Nginx 服务。
>    - `ports: - 80:80`：将容器的 80 端口映射到宿主机的 80 端口，使得外部可以通过宿主机的 80 端口访问 Nginx。
>    - `volumes:`：挂载卷，将宿主机上的目录挂载到容器内部的指定路径。
>      - `/root/nginx/html:/usr/share/nginx/html`：将宿主机上的 `/root/nginx/html` 目录挂载到容器内部的 `/usr/share/nginx/html` 目录，这样 Nginx 就可以访问宿主机上的 HTML 文件。
>      - `/root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf`：将宿主机上的 `/root/nginx/conf/nginx.conf` 文件挂载到容器内部的 `/etc/nginx/nginx.conf`，这样 Nginx 将使用宿主机上的配置文件来配置自身。
>    - `privileged: true`：设置容器的特权模式，为 `true`，这个标志允许容器访问主机的所有设备，并且忽略 SELinux 安全设置。这在解决 Nginx 文件调用的权限问题时可能有所帮助。
> 4. `mysql:`：定义了一个 MySQL 数据库服务。
>    - `image: mysql:5.7.27`：使用的 MySQL 镜像版本。
>    - `ports: - 3306:3306`：将容器的 3306 端口映射到宿主机的 3306 端口，使得外部可以通过宿主机的 3306 端口访问 MySQL。
>    - `environment: - MYSQL_ROOT_PASSWORD=admin`：设置了 MySQL 数据库的环境变量，指定了 root 用户的密码为 "admin"。
> 5. `redis:`：定义了一个 Redis 服务。
>    - `image: redis:latest`：使用的 Redis 镜像版本。
> 6. `vueblog:`：定义了一个 VueBlog 服务。
>    - `image: vueblog:latest`：使用的 VueBlog 镜像版本。
>    - `build: .`：表示使用当前目录下的 Dockerfile 来构建镜像。
>    - `ports: - 8081:8081`：将容器的 8081 端口映射到宿主机的 8081 端口，使得外部可以通过宿主机的 8081 端口访问 VueBlog。
>    - `depends_on: - mysql - redis`：表示 VueBlog 依赖于 mysql 和 redis 服务。这个标志告诉 Docker Compose 在启动 VueBlog 服务之前先启动 mysql 和 redis 服务。注意，这里虽然指定了依赖关系，但并不代表 VueBlog 内部一定能够连接到 mysql 和 redis 服务，还需要在应用内部确保连接设置正确。

nginx中我们对nginx的放置静态资源的html文件夹和配置文件`nginx.conf`进行了一个挂载，所以我们打包后的文件放置到宿主机**/root/nginx/html**文件目录就行了哈

#### 修改application-pro.yml

再回头看看`application-pro.yml`文件，`mysql`和`redis`的链接之前还是`localhost`，现在我们需要修改成容器之间的调用，如何知道mysql和redis的链接地址呢？`docker-compose`就帮我们解决了这个问题，我们可以使用镜像容器的服务名称来表示链接。比如`docker-compose.yml`中`mysql`的服务名称就叫`mysql`、`redis`就叫`redis`。

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/dbd9201e9a64423582e245a9d0d001cf.png)

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/28a935f2f8854759bead5d5ca62a2a11.png)



### 服务器操作

创建`nginx`目挂载目录

```bash
# 创建挂载目录
mkdir -p /root/nginx/conf
mkdir -p /root/nginx/log
mkdir -p /root/nginx/html
```

编写nginx配置文件`root/nginx/conf/nginx.conf`

```bash
vi nginx.conf


#user  root;
worker_processes  1;
events {
  worker_connections  1024;
}
http {
  include       mime.types;
  default_type  application/octet-stream;
  sendfile        on;
  keepalive_timeout  65;
  server {
      listen       80;
      server_name  localhost;
      location / {
          root   /usr/share/nginx/html;
          try_files $uri $uri/ /index.html last; # 别忘了这个哈
          index  index.html index.htm;
      }
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   html;
      }
  }
}
```

上传前端,修改前端访问后端接口配置

```js
axios.defaults.baseURL = "http://XXX.XX.XX.XXX:8081"
```

将前端打包文件`dist`目录通过服务器远程连接工具(Xshell等),存入`/root/nginx/html`目录下

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/4a30914758cf4e2f84548d7631980960.png)

部署后端,打包后端,生成`vueblog-0.0.1-SNAPSHOT.jar`,将`vueblog-0.0.1-SNAPSHOT.jar`,`Dockerfile`,`docker-compose.yml`上传到同一目录下

```bash
[root@localhost root]# ll
total 53096
-rwxr-xr-x.  1 root root      508 Apr 22 02:50 docker-compose.yml
-rw-r--r--.  1 root root      137 Apr 22  2024 Dockerfile
drwxr-xr-x.  5 root root       41 Apr 22 03:02 nginx
-rwxr-xr-x.  1 root root 54354053 Apr 22 03:09 retwisapi-0.0.1-SNAPSHOT.jar
```

执行编排命令`docker-compose up -d`

```bash
# 开始编排
docker-compose up -d
```

其中-d表示后台服务形式启动

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/fd9b5b1103ff408b8a7aaae79116df25.png)

然后我们稍等片刻，特别是开始**Building vueblog**的时候可能时间有点长，耐心等待即可！

最后提示如下：

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/919cf60d670f43198c8c3f43d512fa89.png)

说明我们已经成功编排啦。nginx是80端口，所以我们直接输入ip地址,`http://XXX.XX.XXX.XX:8081`获取访问页

![图片](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/f62108a4241e4b22a8efb9f9db89e963.png)

> 通过`docker-compose`编排成功后,会在docker容器内自动下载镜像,以及按照配置要求生成并启动容器./
>
> 可以通过`docker ps`查看已启动的容器.
>
> ![image-20240422225804486](Docker+nginx%E9%83%A8%E7%BD%B2Springboot+vue%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E9%A1%B9%E7%9B%AE.assets/image-20240422225804486.png)

如果容器启动失败,可以通过`docker logs -f 容器id`查看容器日志,判断错误原因.