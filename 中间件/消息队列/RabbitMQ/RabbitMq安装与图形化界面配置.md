# RabbitMq安装与图形化界面配置

## Linux安装

### 01、概述

官网：https://www.rabbitmq.com/
什么是RabbitMQ,官方给出来这样的解释：

> RabbitMQ is the most widely deployed open source message broker.
> With tens of thousands of users, RabbitMQ is one of the most popular open source message brokers. From T-Mobile to Runtastic, RabbitMQ is used worldwide at small startups and large enterprises.
> RabbitMQ is lightweight and easy to deploy on premises and in the cloud. It supports multiple messaging protocols. RabbitMQ can be deployed in distributed and federated configurations to meet high-scale, high-availability requirements.
> RabbitMQ runs on many operating systems and cloud environments, and provides a wide range of developer tools for most popular languages.
> 翻译以后：
> RabbitMQ是部署最广泛的开源消息代理。
> RabbitMQ拥有成千上万的用户，是最受欢迎的开源消息代理之一。从T-Mobile 到Runtastic，RabbitMQ在全球范围内的小型初创企业和大型企业中都得到使用。
> RabbitMQ轻巧，易于在内部和云中部署。它支持多种消息传递协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模，高可用性的要求。
> RabbitMQ可在许多操作系统和云环境上运行，并为大多数流行语言提供了广泛的开发人员工具。

简单概述：
RabbitMQ是一个开源的遵循AMQP协议实现的基于Erlang语言编写，支持多种客户端（语言）。用于在分布式系统中存储消息，转发消息，具有高可用，高可扩性，易用性等特征。

### 02、安装RabbitMQ

1：下载地址：https://www.rabbitmq.com/download.html
2：环境准备：CentOS7.x+ / Erlang
RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境，第一步就是安装Erlang。

> erlang和RabbitMQ版本的按照比较: https://www.rabbitmq.com/which-erlang.html

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011812781.png)

### 03、 Erlang安装

查看系统版本号

```bash
[root@iZm5eauu5f1ulwtdgwqnsbZ ~]# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 8.3.2011
Release:        8.3.2011
Codename:       n/a
```

#### 3-1:安装下载

参考地址：https://www.erlang-solutions.com/downloads/

```bash
wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
```

#### 3-2：安装成功

```bash
yum install -y erlang
```

#### 3-3：安装成功

```bash
erl -v
```

### 04、安装socat

```bash
yum install -y socat
```

### 05、安装rabbitmq

下载地址：https://www.rabbitmq.com/download.html

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011812789.png)

#### 5-1:下载rabbitmq

```bash
> wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.13/rabbitmq-server-3.8.13-1.el8.noarch.rpm
> rpm -Uvh rabbitmq-server-3.8.13-1.el8.noarch.rpm
```

#### 5-2:启动rabbitmq服务

```bash
# 启动服务
> systemctl start rabbitmq-server
# 查看服务状态
> systemctl status rabbitmq-server
# 停止服务
> systemctl stop rabbitmq-server
# 开机启动服务
> systemctl enable rabbitmq-server
```

### 06、RabbitMQ的配置

RabbitMQ默认情况下有一个配置文件，定义了RabbitMQ的相关配置信息，默认情况下能够满足日常的开发需求。如果需要修改需要，需要自己创建一个配置文件进行覆盖。
参考官网：
1:https://www.rabbitmq.com/documentation.html
2:https://www.rabbitmq.com/configure.html
3:https://www.rabbitmq.com/configure.html#config-items
4：https://github.com/rabbitmq/rabbitmq-server/blob/add-debug-messages-to-quorum_queue_SUITE/docs/rabbitmq.conf.example

### 06-1、相关端口

> 5672:RabbitMQ的通讯端口
> 25672:RabbitMQ的节点间的CLI通讯端口是
> 15672:RabbitMQ HTTP_API的端口，管理员用户才能访问，用于管理RabbitMQ,需要启动Management插件。
> 1883，8883：MQTT插件启动时的端口。
> 61613、61614：STOMP客户端插件启用的时候的端口。
> 15674、15675：基于webscoket的STOMP端口和MOTT端口

一定要注意：RabbitMQ 在安装完毕以后，会绑定一些端口，如果你购买的是阿里云或者腾讯云相关的服务器一定要在安全组中把对应的端口添加到防火墙。

## Docker安装

### 1、虚拟化容器技术—Docker的安装

```bash
（1）yum 包更新到最新
> yum update
（2）安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
> yum install -y yum-utils device-mapper-persistent-data lvm2
（3）设置yum源为阿里云
> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
（4）安装docker
> yum install docker-ce -y
（5）安装后查看docker版本
> docker -v
 (6) 安装加速镜像
 sudo mkdir -p /etc/docker
 sudo tee /etc/docker/daemon.json <<-'EOF'
 {
  "registry-mirrors": ["https://0wrdwnn6.mirror.aliyuncs.com"]
 } 
 EOF
 sudo systemctl daemon-reload
 sudo systemctl restart docker
```

### 2、docker的相关命令

```bash
# 启动docker：
systemctl start docker
# 停止docker：
systemctl stop docker
# 重启docker：
systemctl restart docker
# 查看docker状态：
systemctl status docker
# 开机启动：  
systemctl enable docker
systemctl unenable docker
# 查看docker概要信息
docker info
# 查看docker帮助文档
docker --help
```

### 3、安装rabbitmq

参考网站：
1：https://www.rabbitmq.com/download.html
2：https://registry.hub.docker.com/_/rabbitmq/

#### 1-4、获取rabbit镜像：

```bash
docker pull rabbitmq:management
```

#### 1-5、创建并运行容器

```bash
docker run -di --name=myrabbit -p 15672:15672 rabbitmq:management
```

—hostname：指定容器主机名称
—name:指定容器名称
-p:将mq端口号映射到本地
或者运行时设置用户和密码

```bash
docker run -di --name myrabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management
```

查看日志

```bash
docker logs -f myrabbit
```

#### 1-6、容器运行正常

使用 `http://你的IP地址:15672` 访问rabbit控制台

### 02、额外Linux相关排查命令

```bash
> more xxx.log  查看日记信息
> netstat -naop | grep 5672 查看端口是否被占用
> ps -ef | grep 5672  查看进程
> systemctl stop 服务
```

## Windows 安装

### 安装前提

RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境，第一步就是安装Erlang。

这里这里我要安装的RabbitMQ的版本是 3.9.8 ，此版本至少需要 Erlang 23.2，并支持发布时最新的 Erlang 24 版本 24.1.2。64位

> Installing on Windows — RabbitMQ https://www.rabbitmq.com/install-windows.html#installer
>
> Erlang/OTP版本树 https://erlang.org/download/otp_versions_tree.html

### 安装Erlang

#### 安装

选择安装路径,之后一路next

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813956.png)

#### 配置环境变量

我的电脑->属性->高级系统设置->环境变量

增加系统变量ERLANG_HOME

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813281.png)

增加path路径,`%ERLANG_HOME%\bin`

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813388.png)

#### 测试

打开命令提示符,出现以下安装成功

```bash
erl
```

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813047.png)

### 安装RabbitMq

[Installing on Windows — RabbitMQ](https://www.rabbitmq.com/install-windows.html)

点击下载

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813042.png)

#### 安装

选择安装地址,一路next

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813180.png)

#### 配置环境变量

我的电脑->属性->高级系统设置->环境变量

增加系统变量RABBITQM_SERVER

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011813575.png)

设置path变量

%RABBITQM_SERVER%\sbin,这里是sbin,不是bin

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011812210.png)

#### 测试

打开命令提示符win+R cmd,输入rabbitmqctl status,出现以下界面,证明安装配置成功

![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011812920.png)

## RabbitMQWeb管理界面及授权操作

### 01、RabbitMQ管理界面

#### 01-1：默认情况下，rabbitmq是没有安装web端的客户端插件，需要安装才可以生效

```bash
rabbitmq-plugins enable rabbitmq_management
```

> 说明：rabbitmq有一个默认账号和密码是：`guest` 默认情况只能在localhost本机下访问，所以需要添加一个远程登录的用户。

#### 01-2：安装完毕以后，重启服务即可

```bash
systemctl restart rabbitmq-server
```

> 一定要记住，在对应服务器(阿里云，腾讯云等)的安全组中开放`15672`的端口。

#### 01-3：在浏览器访问

http://ip:15672/ 如下：
![img](RabbitMq%E5%AE%89%E8%A3%85%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%E9%85%8D%E7%BD%AE.assets/202303011752270.png)

### 02、授权账号和密码

#### 2-1：新增用户

```bash
# rabbitmqctl add_user 账号 密码
rabbitmqctl add_user admin admin
```

#### 2-2:设置用户分配操作权限

```bash
# rabbitmqctl set_user_tags 账号 administrator
rabbitmqctl set_user_tags admin administrator
```

用户级别：

- 1、administrator 可以登录控制台、查看所有信息、可以对rabbitmq进行管理
- 2、monitoring 监控者 登录控制台，查看所有信息
- 3、policymaker 策略制定者 登录控制台,指定策略
- 4、managment 普通管理员 登录控制台

#### 2-3：为用户添加资源权限

```bash
rabbitmqctl.bat set_permissions -p / admin ".*" ".*" ".*"
```

## 03、小结：

```bash
rabbitmqctl add_user 账号 密码
rabbitmqctl set_user_tags 账号 administrator
rabbitmqctl change_password Username Newpassword 修改密码
rabbitmqctl delete_user Username 删除用户
rabbitmqctl list_users 查看用户清单
rabbitmqctl set_permissions -p / 用户名 ".*" ".*" ".*" 为用户设置administrator角色
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
```