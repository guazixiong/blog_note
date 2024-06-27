# linux centos7安装zookeeper

## 下载

可以去官网下载，然后上传到 linux. 

```bash
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```

如果没有安装wget会报错，需要先安装  `yum -y install wget`

>  zookeeper下载会有点慢。。。。。建议下载后上传

## 配置启动

```bash
# 解压
tar -zxvf zookeeper-3.4.6.tar.gz
# 进入解压出来的目录
cd zookeeper-3.4.6/

# 不介意zookeeper数据存储的以下不执行
# 创建保存数据的目录
mkdir data
# 进入data目录
cd data
# 查看当前目录(复制,用于配置zookeeper数据的保存)
pwd
# 返回上一级目录
cd ..
# 生成自定义配置文件
 cp ./conf/zoo_sample.cfg ./conf/zoo.cfg
# 进入配置文件目录
cd conf
# 编辑配置文件
vim zoo.cfg

### ......  
# 修改dataDir目录为自定义data位置
dataDir=/usr/local/zookeeper-3.4.6/zookeeper-3.4.6/data
### ......

## wq退出保存

# 返回上一层(进入zookeeper-3.4.6根目录)
cd ..
```

> 由于zookeeper数据的保存，它默认的位置是临时的，会被定时清除。因此手动配置存储目录

## 启动zookeeper

```bash
# 进入/zookeeper-3.4.6/bin目录
cd bin
# 启动zookeeper
[root@VM-8-7-centos bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/zookeeper-3.4.9/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

# 查看状态
[root@VM-8-7-centos bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: standalone
```

> 出现error选项,发现zookeeper并未启动,是由于没有安装jdk
>
> ```bash
> yum -y install java-1.8.0-openjdk.x86_64
> ```
>
> 重新启动zookeeper
>
> ```bash
> # 启动zookeeper
> [root@VM-8-7-centos bin]# ./zkServer.sh start
> 
> # 查看状态
> [root@VM-8-7-centos bin]# ./zkServer.sh status
> ```

