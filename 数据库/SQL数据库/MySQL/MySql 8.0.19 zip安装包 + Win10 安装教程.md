# MySql 8.0.19 安装

[toc]

## 1. zip安装包下载

推荐去官网[MySql](https://dev.mysql.com/downloads/mysql/)下载，根据电脑系统选择合适的版本。

## 2.Windows环境配置

下载之后解压到安装目录下，例如` D:\mysql-8.0.19-winx64`。
然后找到**控制面板–>系统和安全–>系统–>高级系统设置–>环境变量–>系统变量**

![在这里插入图片描述](MySql%208.0.19%20zip%E5%AE%89%E8%A3%85%E5%8C%85%20+%20Win10%20%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.assets/202303261202374.jpeg)

系统变量中添加两个，第一个直接添加Mysql_Home，第二个添加到Path里（注意第二个是/bin文件夹目录）。

![在这里插入图片描述](MySql%208.0.19%20zip%E5%AE%89%E8%A3%85%E5%8C%85%20+%20Win10%20%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.assets/202303261202344.jpeg)

![在这里插入图片描述](MySql%208.0.19%20zip%E5%AE%89%E8%A3%85%E5%8C%85%20+%20Win10%20%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.assets/202303261202150.jpeg)

## 3. ini配置文件修改

![在这里插入图片描述](MySql%208.0.19%20zip%E5%AE%89%E8%A3%85%E5%8C%85%20+%20Win10%20%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.assets/202303261202478.jpeg)

刚解压的文件夹中没有选中的两个文件，其中data文件夹在后面初始化时会自动生成，my.ini文件需要自己创建。先创建 .txt文件，添加下面一段进去，再重命名就可以。标明需要改的两个地方要改成自己的路径。

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir= D:\mysql-8.0.19-winx64       (需要改）
# 设置mysql数据库的数据的存放目录
datadir= D:\mysql-8.0.19-winx64\data （需要改）
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

## 4.初始化以及安装

以**管理员身份**打开[cmd](https://so.csdn.net/so/search?q=cmd&spm=1001.2101.3001.7020)，输入 **mysqld --initialize --console** 命令进行初始化。
在输出中有一行是：

```
[Note] [MY-010454] [Server] A temporary password is generated for root@localhost:**********
```

**其中root@localhost:后面的“**”就是初始密码，初次登陆时要用到。**

接下来 **mysqld --install** 命令行安装。遇到问题可以看第6步是否有解决办法。

## 5.启动和修改密码

```bash
# 启动mysql
net start mysql

# 登录mysql
mysql -u root -p

# 修改密码 
ALTER USER USER() IDENTIFIED BY 'NewPassword';

# 退出mysql
quit

# 关闭服务
net stop mysql
```

![img](MySql%208.0.19%20zip%E5%AE%89%E8%A3%85%E5%8C%85%20+%20Win10%20%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.assets/202303261204622.jpeg)