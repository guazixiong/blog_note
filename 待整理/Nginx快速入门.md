# Nginx快速入门

## 公司产品出现瓶颈？

我们公司项目刚刚上线的时候，并发量小，用户使用的少，所以在低并发的情况下，一个jar包启动应用就够了，然后内部tomcat返回内容给用户。
![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146145498-450cfea4-5849-4604-89be-cb76767e51b3.png)
但是慢慢的，使用我们平台的用户越来越多了，并发量慢慢增大了，这时候一台服务器满足不了我们的需求了。
![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146145564-e703aa1c-58fc-42ea-8edb-241dd4f6b655.png)
于是我们横向扩展，又增加了服务器。这个时候几个项目启动在不同的服务器上，用户要访问，就需要增加一个代理服务器了，通过代理服务器来帮我们转发和处理请求。
![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146145588-47ac9e82-d40e-4d56-b98e-00a31b16bd6b.png)
我们希望这个代理服务器可以帮助我们接收用户的请求，然后将用户的请求按照规则帮我们转发到不同的服务器节点之上。这个过程用户是无感知的，用户并不知道是哪个服务器返回的结果，我们还希望他可以按照服务器的性能提供不同的权重选择。保证最佳体验！所以我们使用了Nginx。

## 什么是Nginx？

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。2011年6月1日，nginx 1.0.4发布。

其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。在全球活跃的网站中有12.18%的使用比率，大约为2220万个网站。

Nginx 是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的服务。Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够不间断服务的情况下进行软件版本的升级。

Nginx代码完全用C语言从头写成。官方数据测试表明能够支持高达 50,000 个并发连接数的响应。

## Nginx作用？

Http代理，反向代理：作为web服务器最常用的功能之一，尤其是反向代理。

正向代理 (如VPN)

通过代理去访问服务器资源,服务器与代理交互,返回数据再与客户端进行交互,对于客户端而言,知道请求的是什么服务器,但服务器只知道是通过代理服务器请求.
![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146145608-abd07c33-ead5-44f4-88b2-45f8df313b82.png)
反向代理 (nginx)

客户端通过云服务,净由代理去访问服务器,这是代理充当转发,客户端不明白具体是那个服务器进行处理,只知道自己请求了云服务,而服务器知道是来自那个客户端的请求.
![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146145939-d9578831-e718-4cae-8e91-883e1589687c.png)

## 负载均衡策略

### 内置策略

- 轮询
- 加权
- IP hash



#### 轮询(Round Robin)

在轮询模式下，Nginx将请求按照顺序分配给后端服务器。每个请求都会被依次发送到下一个服务器，循环往复。

假设有三个后端服务器：A、B、C。依次按照顺序接收请求。

- 请求1 -> 后端A
- 请求2 -> 后端B
- 请求3 -> 后端C
- 请求4 -> 后端A
- 请求5 -> 后端B
- ...

这个过程将一直循环。

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146033-80a7cab0-22c1-4e30-8170-cc853bcdc98f.png)



加权(Weight Round Robin)

在加权轮询模式下，可以为每个后端服务器分配一个权重，以便某些服务器获得更多的请求。(权重越高,请求越高)

我们通常可以根据不同服务器的配置情况,设置不同的权重

假设有三个后端服务器：A（权重3）、B（权重2）、C（权重1）。

- 请求1 -> 后端A
- 请求2 -> 后端A
- 请求3 -> 后端A
- 请求4 -> 后端B
- 请求5 -> 后端B
- 请求6 -> 后端C
- 请求7 -> 后端A
- ...

权重决定了每个服务器接收请求的频率。

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146056-53e49f61-8b55-4639-978a-74f8817376f1.png)



IP hash

IP哈希根据客户端的IP地址将请求定向到特定的后端服务器。这对于确保特定客户端的所有请求都由相同的服务器处理很有用,可以解决session不共享的问题。

假设有三个后端服务器：A、B、C。根据客户端的IP地址，将请求定向到特定的后端服务器。

- 对于IP地址X，始终发送到后端A
- 对于IP地址Y，始终发送到后端B
- 对于IP地址Z，始终发送到后端C
- ...

这样，相同IP地址的客户端的所有请求都将被路由到相同的后端服务器。

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146038-8960a32b-c3e6-4090-8d00-73d1d4c7d5a9.png)



### 扩展策略

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。扩展策略，就天马行空，只有你想不到的没有他做不到的。

轮询



### 动静分离

动静分离，在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件），这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源响应的速度。

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146070-ecb6a755-b70e-440e-bd51-38877a2f5482.png)

目前，通过使用Nginx大大提高了我们网站的响应速度，优化了用户体验，让网站的健壮性更上一层楼！

# Nginx的安装

## windows下安装

1、下载nginx

http://nginx.org/en/download.html 下载稳定版本。
以nginx/Windows-1.16.1为例，直接下载 nginx-1.16.1.zip。
下载后解压，解压后如下：

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146442-e7499331-bfe2-4160-80b3-073c79f296f1.png)

2、启动nginx

- 直接双击nginx.exe，双击后一个黑色的弹窗一闪而过
- 打开cmd命令窗口，切换到nginx解压目录下，输入命令 `nginx.exe` ，回车即可

3、检查nginx是否启动成功

直接在浏览器地址栏输入网址 [http://localhost:80](http://localhost/) 回车，出现以下页面说明启动成功！

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146604-f99f6d63-b077-4ea7-b0c0-4898da0ec2ac.png)

4、配置监听

nginx的配置文件是conf目录下的nginx.conf，默认配置的nginx监听的端口为80，如果80端口被占用可以修改为未被占用的端口即可。

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146550-d7c048cf-dacf-49ac-8345-c34faad0e20a.png)

当我们修改了nginx的配置文件nginx.conf 时，不需要关闭nginx后重新启动nginx，只需要执行命令 nginx -s reload 即可让改动生效

5、关闭nginx

如果使用cmd命令窗口启动nginx， 关闭cmd窗口是不能结束nginx进程的，可使用两种方法关闭nginx

(1)输入nginx命令 `nginx -s stop`(快速停止nginx) 或 `nginx -s quit`(完整有序的停止nginx)

(2)使用`taskkill`命令 `taskkill /f /t /im nginx.exe`

```plain
taskkill 是用来终止进程的，
/f  强制终止 .
/t  终止指定的进程和任何由此启动的子进程。
/im 指定的进程名称 .
```

## linux下安装

1、安装gcc

安装 `nginx` 需要先将官网下载的源码进行编译，编译依赖 `gcc` 环境，如果没有 `gcc` 环境，则需要安装：

```plain
yum install gcc-c++
```

2、PCRE pcre-devel 安装

`PCRE`(Perl Compatible Regular Expressions) 是一个`Perl`库，包括 `perl` 兼容的正则表达式库。`nginx` 的 `http`模块使用 `pcre` 来解析正则表达式，所以需要在 `linux` 上安装 `pcre` 库，`pcre-devel` 是使用`pcre`开发的一个二次开发库。`nginx`也需要此库。命令：

```plain
yum install -y pcre pcre-devel
```

3、zlib 安装

zlib 库提供了很多种压缩和解压缩的方式， `nginx` 使用 `zlib` 对 `http` 包的内容进行 `gzip` ，所以需要在 `Centos` 上安装 `zlib` 库。

```plain
yum install -y zlib zlib-devel
```

4、OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```plain
yum install -y openssl openssl-devel
```

5、下载安装包

手动下载.tar.gz安装包，地址：https://nginx.org/en/download.html

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146636-f3eda14b-1cdb-4033-a16e-b8f3c5d911a2.png)

下载完毕上传到服务器上 /root

6、解压

```bash
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
```

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146611-e38e77d2-d43e-4830-a051-5c930e239b94.png)

7、配置

使用默认配置，在nginx根目录下执行

```bash
./configure
make
make install
```

查找安装路径： whereis nginx

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146146832-ed4de016-cca5-4c27-bd35-14bac393db49.png)

## Nginx常用命令

```plain
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程
```

启动成功访问 服务器ip:80

![img](Nginx%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/1700146147045-73e302d2-9d87-47fd-8603-0d2776e75a8a.png)

注意：如何连接不上，检查阿里云安全组是否开放端口，或者服务器防火墙是否开放端口！
相关命令：

```plain
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
# 查看防火墙规则
firewall-cmd --list-all
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```

# 演示

```plain
upstream lb{
    server 127.0.0.1:8080 weight=1;
    server 127.0.0.1:8081 weight=1;
}

location / {
    proxy_pass http://lb;
}
```

## 引申

开发过程中会用到Session,由于进行了nginx分发请求,导致session无法共享,我们通常使用的是redis来解决相关问题.





需拓展: 

<details class="lake-collapse"><summary id="u2ed848ce"><span class="ne-text">轮询（Round Robin）：</span></summary><pre data-language="bash" id="hR6mc" class="ne-codeblock language-bash" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
</code></pre></details>

<details class="lake-collapse"><summary id="u711b3e97"><span class="ne-text">加权轮询</span></summary><pre data-language="bash" id="DjodN" class="ne-codeblock language-bash" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>http {
    upstream backend {
        server backend1.example.com weight=3;
        server backend2.example.com weight=2;
        server backend3.example.com weight=1;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
</code></pre></details>

<details class="lake-collapse"><summary id="u02888d46"><span class="ne-text">IP Hash</span></summary><pre data-language="bash" id="Ie6pM" class="ne-codeblock language-bash" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>http {
    upstream backend {
        ip_hash;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
</code></pre></details>

<details class="lake-collapse"><summary id="ubd182cb6"><span class="ne-text">拓展策略</span></summary><span id="l912o"></span></details>

<details class="lake-collapse"><summary id="u10d8ef52"><span class="ne-text">Nginx 配置文件描述</span></summary><p id="u6019871b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"></p></details>

<details class="lake-collapse"><summary id="ue0844bb6"><span class="ne-text">Nginx 动静分离</span></summary><p id="u868181b7" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"></p></details>

<details class="lake-collapse"><summary id="u805782db"><span class="ne-text">Nginx 配置https</span></summary><p id="u5f46a220" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"></p></details>