# 一台服务器(虚拟机)构建Mysql主从复制

## 准备前提

+ 一台服务器 / 虚拟机
+ 安装docker环境
+ 防火墙开放对外端口号`33061`和`33062`(一主一从,多主多从,开放多个)

## 构建主从复制

基于mysql8镜像,创建主从库`mysql8_master`和`mysql8_slave`

```bash
[root@lavm-googc8m29h slave]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                                                    NAMES
ebe768e56fbf   mysql          "docker-entrypoint.s…"   About a minute ago   Up About a minute   33060/tcp, 0.0.0.0:33062->3306/tcp, :::33062->3306/tcp   mysql8_slave
cf052c62a815   mysql          "docker-entrypoint.s…"   About a minute ago   Up About a minute   33060/tcp, 0.0.0.0:33061->3306/tcp, :::33061->3306/tcp   mysql8_master
```

### 构建主库

#### 创建docker容器和远程访问用户

```bash
# 创建主机mysql挂载目录
sudo mkdir -p /usr/local/mysql8
sudo mkdir -p /usr/local/mysql8/master/conf
sudo mkdir -p /usr/local/mysql8/master/data
# 创建主机自定义配置文件myconf.cnf
cd /usr/local/mysql8/master
chmod 777 conf
cd conf
touch myconf.cnf 
# 创建容器 映射33061目录,挂载自定义配置文件和数据,初始化root用户密码
docker run --name mysql8_master -p 33061:3306 \
-v /usr/local/mysql8/master/conf:/etc/mysql/conf.d \
-v /usr/local/mysql8/master/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root -d mysql
# 进入容器
docker exec -it mysql8_master bash
# 登录mysql
mysql -u root -p

# 修改加密规则
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';

# 添加远程登录用户
create user 'pd_slave'@'%' identified by '123456';
# 修改加密规则
ALTER USER 'pd_slave'@'%' IDENTIFIED WITH mysql_native_password BY 'pd_slave';
# 授予该用户对所有数据库的所有权限
GRANT ALL PRIVILEGES ON *.* TO 'pd_slave'@'%';

# 刷新权限（可以直接复制）
FLUSH PRIVILEGES;
```

> + `Authentication plugin 'caching_sha2_password' cannot be loaded` : 修改mysql8的加密规则,能够通过Navicat远程连接数据库
> + `create user 'pd_slave'@'%' identified by '123456';` 创建`pd_slave`用户,使得后续从库通过远程用户获取`binary_log`

#### 配置文件

修改主mysql配置文件,`conf\my.cnf`,写入以下参数

```bash
[mysqld]
# 服务器唯一id,默认值为1
server-id=1
# 设置日志格式,默认值为ROW
binlog_format=STATEMENT
# 二进制日志名,默认binlog
# log-bin=binlog
# 设置需要复制的数据库,默认复制全部数据库
# bin-do-db=mytestdb
# 设置不需要复制的数据库
# bin-ignore-db=mysql
# bin-ignore-db=information_schema
```

+ `binlog_format` : 日志格式
  + `binlog_format=STATEMENT`:日志记录的是主机数据库的写指令,性能高,但是now()之类的函数以及获取系统参数的操作会出现主从数据不同步的问题.
  + `binlog_format=ROW(默认)`:日志记录的是主机数据库的写后的数据,批量操作时性能交叉,解决now()或者user()或者@@hostname等操作在主从机器上不一致的问题
  + `binlog_format=MIXED`: 是以上两种level的混合使用,有函数用ROW,没函数用STATEMENT,但是无法识别系统变量

#### 重启主机mysql.

```bash
docker restart mysql8_master
```

#### 查看主机状态

主机查询master状态,执行完此步骤后不要再操作主服务器mysql,防止主服务器状态值变化.

> 从机需要根据master的`file`和`position`,

```bash
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |      860 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.01 sec)
```

### 构建从库

#### 创建docker容器

```bash
# 创建主机mysql挂载目录
sudo mkdir -p /usr/local/mysql8
sudo mkdir -p /usr/local/mysql8/slave/conf
sudo mkdir -p /usr/local/mysql8/slave/data
# 创建主机自定义配置文件myconf.cnf
cd /usr/local/mysql8/slave
chmod 777 conf
cd conf
touch myconf.cnf 
# 创建容器 映射33061目录,挂载自定义配置文件和数据,初始化root用户密码
docker run --name mysql8_slave -p 33061:3306 \
-v /usr/local/mysql8/slave/conf:/etc/mysql/conf.d \
-v /usr/local/mysql8/slave/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root -d mysql
# 进入容器
docker exec -it mysql8_slave bash
# 登录mysql
mysql -u root -p

# 修改加密规则
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';

# 刷新权限（可以直接复制）
FLUSH PRIVILEGES;
```

#### 配置文件

修改`conf\my.cnf`

```bash
[mysqld]
# 服务器唯一id,每台服务器的id必须不同,如果配置其他从机,注意修改id
server-id=2
# 中继日志名,默认xxxxxxxxxx-relay-bin
# relay-log=relay-bin
```

如果存在其他从机

```bash
[mysqld]
# 服务器唯一id,每台服务器的id必须不同,如果配置其他从机,注意修改id
server-id=3
# 中继日志名,默认xxxxxxxxxx-relay-bin
# relay-log=relay-bin
```

#### 重启从库

```bash
docker restart mysql8_slave
```

#### 配置主从关系

在从库上执行,

获取主库的file和position

```bash
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |      860 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.01 sec)
```

执行以下命令

`CHANGE MASTER TO MASTER_HOST='117.72.32.111',MASTER_USER='pd_slave',MASTER_PASSWORD='123456',MASTER_PORT=33061,MASTER_LOG_FILE='{file}',MASTER_LOG_POS={position};`  替换为自己的`file`和`position`

```bash
root@ebe768e56fbf:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.27 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CHANGE MASTER TO MASTER_HOST='117.72.32.111',MASTER_USER='pd_slave',MASTER_PASSWORD='123456',MASTER_PORT=33061,MASTER_LOG_FILE='binlog.000004',MASTER_LOG_POS=860;
Query OK, 0 rows affected, 9 warnings (0.10 sec)

```

+ 设置主机地址:   `CHANGE MASTER TO MASTER_HOST='117.72.32.111'`
+ 设置主机访问用户账号: `MASTER_USER='pd_slave'`

+ 设置主机访问用户密码: `MASTER_PASSWORD='123456'`

+ 设置主机端口号: `MASTER_PORT=33061`

+ 设置主机binlog: `MASTER_LOG_FILE='binlog.000004'`
+ 设置主机position: `MASTER_LOG_POS=860`

#### 启动从机的主从复制

```bash
# 启动主从同步
mysql> START SLAVE;
Query OK, 0 rows affected, 1 warning (0.02 sec)

# 查看状态(不需要分号) \G表示纵向显示,G需要大写
mysql> SHOW SLAVE STATUS\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 117.72.32.111
                  Master_User: pd_slave
                  Master_Port: 33061
                Connect_Retry: 60
              Master_Log_File: binlog.000004
          Read_Master_Log_Pos: 860
               Relay_Log_File: ebe768e56fbf-relay-bin.000002
                Relay_Log_Pos: 321
        Relay_Master_Log_File: binlog.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 860
              Relay_Log_Space: 537
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: b2ea1cc7-3949-11ef-adf3-0242ac110006
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

ERROR:
No query specified

mysql>

```

两个关键进程: 下面两个参数都是Yes,则说明主从配置成功!

+ `Slave_IO_Running: Yes`

+ `Slave_SQL_Running: Yes`

### 测试主从复制

在主机执行以下sql,在从机中查看数据库、表和数据是否已经被同步

```bash
# 创建db_user数据库
CREATE DATABASE db_user;
# 使用db_user数据库
use db_user;
# 创建测试表`test_table`
CREATE TABLE `test_table` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `col1` varchar(255) DEFAULT NULL,
  `col2` varchar(255) DEFAULT NULL,
  `col3` int(11) DEFAULT NULL,
  `col4` decimal(10,2) DEFAULT NULL,
  `col5` date DEFAULT NULL,
  `col6` datetime DEFAULT NULL,
  `col7` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `col8` tinyint(1) DEFAULT NULL,
  `col9` char(10) DEFAULT NULL,
  `col10` text,
  `col11` blob,
  `col12` mediumtext,
  `col13` enum('val1','val2','val3') DEFAULT NULL,
  `col14` set('val1','val2','val3') DEFAULT NULL,
  `col15` smallint(6) DEFAULT NULL,
  `col16` bigint(20) DEFAULT NULL,
  `col17` float DEFAULT NULL,
  `col18` double DEFAULT NULL,
  `col19` year(4) DEFAULT NULL,
  `col20` time DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
# 创建测试数据
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (1, 'value_1', 'data_1', 1, 0.01, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 1, 'fd947ccb-3', 'texttexttexttexttexttexttexttexttexttext', 0xAB785C069FC049D130ECBECC0A3F106F2289B684ACFA01B0F58F862FB719F5E29DECC3E92EE6CF1F887DC94AF52A202012D3E6AE570F99948F3523813FD63D280D215646412BDE24DF33D9D57D1DEA7633D8B4449186CA615A6E2B9053AB55D839F378B3, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 1, 1000, 0.1, 0.001, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (2, 'value_2', 'data_2', 2, 0.02, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 0, 'fd94cad1-3', 'texttexttexttexttexttexttexttexttexttext', 0x68FB9C5B47F19164DA91B1EDE5ECC414612C73FA492A82D82E296FF2C0D86CB6F9A9496F4034FC273F70F040CF5D90FE2EA961EAA570191185F8DD18DC90C7DF70A1773989C96444BB18BDF2F50DC4F4F07661434CB84DADCBE0099EC0DCF0CD8F486D55, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 2, 2000, 0.2, 0.002, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (3, 'value_3', 'data_3', 3, 0.03, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 1, 'fd95013c-3', 'texttexttexttexttexttexttexttexttexttext', 0xD359EB7BF9899FA4EF309A69E0CAEBFBBB25BCEECADC73467D6754B372D9158577D07F1B99A5AEE671287A06B843E9A30892EA59DF329EC9AFF39CE3162E256B1B903382FE69B3413D8D7A38BC427D3ACDDC9EEFD0C0809A9DC4E3D1A9B3EF68B4910AC6, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 3, 3000, 0.3, 0.003, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (4, 'value_4', 'data_4', 4, 0.04, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 0, 'fd974453-3', 'texttexttexttexttexttexttexttexttexttext', 0xACC9E718CDE7AB288D10C6126D7BED76F8E8BBBEB489EDE2F87ED0CE3E4454F7F10600A7060A74321BC24898365D8F911D5559E5B11B584391EEEF589632EC212C8F82D4B06A524F4D79C6FCCC47673D962EC4427AA1A9BC70E1CC3E30591839F7AAE484, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 4, 4000, 0.4, 0.004, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (5, 'value_5', 'data_5', 5, 0.05, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 1, 'fd97c8e9-3', 'texttexttexttexttexttexttexttexttexttext', 0x52064D812765BCBDCEF56F6A23CFDE9A5822EB2775A34EAC11A24247D8E8BF56B8DF7966F87A96633E5BA9A366A14834EEE1C07AF4C29866D88436A45A6AEE0E64D366B211F2934726DBE84429A57BB40257B9A436510F5277A15825D5182CB8A244560B, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 5, 5000, 0.5, 0.005, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (6, 'value_6', 'data_6', 6, 0.06, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 0, 'fd9816cf-3', 'texttexttexttexttexttexttexttexttexttext', 0xA4A5C61A45B816C0E32B8715B52FA87834EDEDCD957A5174B98334A57EFDB273AD57F1DFF74223F544396D8CD1B79B0BC7BBCEFBE1DC9FFC4FE2D7745FDE4A6754DBA3482ED5FD8B8568A04839C498DF2B5588811C180F53D80EECA1A02E7C302F6DE804, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 6, 6000, 0.6, 0.006, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (7, 'value_7', 'data_7', 7, 0.07, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 1, 'fd986440-3', 'texttexttexttexttexttexttexttexttexttext', 0x1866D97760F15BCBCCF7A68ACA923D389DD401C1A079A81C41635A5E3890575F2823ABCC6F82AA5035F19F70F28317D294D45B9A1F4E8A4C80E1ED7D92423CE79ABFDD17BA917872467F18C693175145BFF59804D29EDC7913CEFC7D27E41C6647685699, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 7, 7000, 0.7, 0.007, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (8, 'value_8', 'data_8', 8, 0.08, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 0, 'fd9896a8-3', 'texttexttexttexttexttexttexttexttexttext', 0x625CE4D5CCC0F1C8CB997E54C7F8E41D6991ECEF85279966B31411C8C4B65616962B45B9EF98B86AA1D37878FF0C368D1BDFFC9E3B99E3228CCA5DF2A0377E244979151B8318C47CE6066D2AEEF7500DBBEBDD0E131563DAFBBE5135BF025F3FEC742888, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 8, 8000, 0.8, 0.008, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (9, 'value_9', 'data_9', 9, 0.09, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 1, 'fd98db64-3', 'texttexttexttexttexttexttexttexttexttext', 0x9E594F164968506029DE85492ED783A6F11BA1B99D3E5AB5C378F70304BE23B88132EFCE28D5C4686F3EC1AF5BFFA0CEE811CBF539312F176ACDA762FA34152F37FB93D0B7419F72158E0A9B8C9C3FACB87CF71671BDB6B57600A7E37743D90CDAD1B1D0, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 9, 9000, 0.9, 0.009, 2024, '01:45:53');
INSERT INTO `test_table`(`id`, `col1`, `col2`, `col3`, `col4`, `col5`, `col6`, `col7`, `col8`, `col9`, `col10`, `col11`, `col12`, `col13`, `col14`, `col15`, `col16`, `col17`, `col18`, `col19`, `col20`) VALUES (10, 'value_10', 'data_10', 10, 0.10, '2024-06-27', '2024-06-27 01:45:53', '2024-06-27 01:45:53', 0, 'fd991f5e-3', 'texttexttexttexttexttexttexttexttexttext', 0xD47CE11A6DCB74C4E32A45C09E864753DFB6BCC2EA583552DE677AF733C1573C991BE8EEDE37AC17F81F35247BF0A0E112EBE2A7DA2A4943B5A6ABF03AF1401529371E1346D3E9C4C62D0778DBB87A10AA5D8D181EB929B5AA507AED2F9EB56C79D45CEC, 'mediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtextmediumtext', 'val1', 'val2', 10, 10000, 1, 0.01, 2024, '01:45:53');
```

## 问题汇总

### 远程连接提示``Authentication plugin 'caching_sha2_password' cannot be loaded`, `

修改mysql数据库用户登录加密规则

```bash
# 登录mysql
mysql -u root -p
# 修改加密规则
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
# 刷新权限（可以直接复制）
FLUSH PRIVILEGES;
```

### 启动主从同步后,`Slave_IO_Running: No`或者`Connecting`的情况

此时下方的LAST_IO_ERROR错误日志,根据日志中的错误信息搜索解决方案.

