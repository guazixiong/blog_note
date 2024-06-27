# Linux服务器 下载并安装tomcat

下载安装tomcat前，先下载安装JDK

下载tomcat：[下载地址](https://links.jianshu.com/go?to=https%3A%2F%2Ftomcat.apache.org%2Fdownload-90.cgi)

![img](Linux%E5%AE%89%E8%A3%85tomcat.assets/202305272212528.webp)

上传文件，并解压：

```css
tar -zxvf apache-tomcat-9.0.19.tar.gz
```

加入tomcat的环境变量 安装完成后需要配置一下环境变量，编辑/etc/profile文件：

```bash
vi /etc/profile
```

在文件尾部添加如下配置：

```bash
export CATALINA_HOME=/wocloud/tomcat_cluster/tomcat1/apache-tomcat-9.0.19
```

编辑完成后按esc后输入:wq保存退出，最后一步就是通过source命令重新加载/etc/profile文件，使得修改后的内容在当前shell窗口有效：

```bash
source /etc/profile
```

启动tomcat，进入/wocloud/apache-tomcat-9.0.19/bin（我直接安装在了wocloud目录下，如果安装路径不一样，按照自己的路径）

执行：`./startup.sh`，出现如下图片则启动成功。此时还是不能访问的tomcat主页的。

```bash
./startup.sh
```

然后我们需要在阿里云服务器管理中开发8080和8009端口。 这个时候可以用你本机上的浏览器访问一下，如果可以访问，跳过第6步。不能访问进行第6步。

配置防火墙

在CentOS 7中引入了一个更强大的防火墙——Firewall。
打开firewall

```bash
systemctl start firewalld
```

我们需要在Firewall中开启8080和8009端口（这里的两个端口是tomcat默认的访问tomcat服务器的端口，如果自己改变了端口，要按照自己修改的端口），也就是将8080和8009端口加入到zone（Firewall的新特性，简单讲它的作用就是定义了网络区域网络连接的可信等级）中。命令如下:

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=8009/tcp --permanent
```

这样就成功的将8081端口加入了public区域中，permanent参数表示永久生效，即重启也不会失效，最后不要忘记更新防火墙规则：

```bash
firewall-cmd --reload
```

OK，下面看一下public区域下所有已打开的端口，命令如下：

```bash
firewall-cmd --zone=public --list-ports
```

这个时候，就可以通过服务器的外网ip或者域名来访问tomcat了.