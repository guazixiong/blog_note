# Jenkins使用问题汇总

- 容器内部出现`mvn:command not found`的问题
- Jenkins 构建项目提示`mvn command not fount`
- Jenkins 构建时,nohup 命令无法成功运行程序
- jenkins插件安装失败

## 容器内部出现mvn:command not found的问题

原因为`docker`容器内部未挂载`maven`

+ 基于已有的`docker`容器,追加
+ 创建新的`jenkins`容器

运行容器的时候将宿主机的java和maven目录挂载进去

```bash
[root@ jenkins_home]# docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED      STATUS        PORTS                                                                                NAMES
6f7b3143fa97   jenkins/jenkins   "/usr/bin/tini -- /u…"   2 days ago   Up 43 hours   0.0.0.0:8100-8101->8100-8101/tcp, 0.0.0.0:50000->50000/tcp, 0.0.0.0:8082->8080/tcp   jenkins
```

```bash
docker run 
-d -p 8080:8080 \
-v /usr/lib/jvm/java-1.8-openjdk:/usr/lib/jvm/java-1.8-openjdk \
-v /usr/local/maven/maven3:/usr/local/maven/maven3 \
容器id
```

`-v` 是将宿主机的目录挂载到容器内

`:`冒号前面的是宿主机目录，冒号后面的是容器应用的目录

进入jenkins容器内部,修改环境变量配置.

```bash
docker exec -it 容器id /bin/bash
```

```bash
vi /etc/profile
```

> 可能存在vi command not found的提示,是由于容器内部没有vi的命令,我们可以通过docker cp的方式copy自定义的profile文件到docker 容器内部(友情提示,先保存容器内部的profile文件)
>
> ```bash
> docker cp profile 容器id:/etc/ 
> # 覆写`profile`文件
> ```

刷新docker 容器内部全局环境变量,重新执行`mvn`命令,成功

```bash
source /etc/profile
mvn --version
```

## Jenkins 构建项目提示`mvn command not fount`

这是由于jenkins没有我们服务器path的环境变量,所以出现`command not found`的错误

解决方案:

在Jenkins部署服务器输出一下PATH路径：

```bash
[root@iZbp192vpw33333rwntdtbfrZ ~]# echo $PATH
/usr/local/maven/apache-maven-3.9.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/java/jdk1.8.0_361/bin:/usr/local/java/jdk1.8.0_361/jre/bin:/usr/local/git/git-2.20.0/bin:/usr/local/node/node-v14.17.0-linux-x64/bin:/root/bin
```

然后在**Jenkins -> Manager Jenkins -> Configure System**中添加环境变量：

![img](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307231017898.jpeg)

找到 Global properties，勾选中Environment variables，一个**PATH**变量以后保存(注：PATH是大写哦)

![img](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307231017064.jpeg)

重新构建即可.

## Jenkins 构建时,nohup 命令无法成功运行程序

> **当Jenkins 执行nohup 命令时 Jenkins程序 只负责运行 伪命令行 nuhup 命令，并不保证是否成功运行 java 程序。**

解决方法:

1. 使用字符串函数 获取 buile_id 配合 dontKillMe 来保护 nohup执行程序不被Jenkins 立即 kill掉。
   在jenkins的部署脚本里最上面写上

   ```bash
   [code=java]
   BUILD_ID=dontKillMe
   nohup java -jar xxx.jar
   ```

2. 在nohup 命令下发 添加sleep5 ，使用休眠方式来保证 nohup命令 能真正完成执行成功。

## 如何更新Jenkins版本?

1. 下载最新的`jenkins.war` 

Jenkins download and deployment: https://www.jenkins.io/download/

![image-20230723104238243](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307231042340.png)

2. 将最新的`jenkins.war`上传至服务器

![image-20230702125356640](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021253680.png)

3. 获取`docker 容器`,将`jenkins.war`更新至容器内部

获取`docker`容器,容器id`6f7b3143fa97`

```bash
[root@ jenkins_home]# docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED      STATUS        PORTS                                                                                NAMES
6f7b3143fa97   jenkins/jenkins   "/usr/bin/tini -- /u…"   2 days ago   Up 43 hours   0.0.0.0:8100-8101->8100-8101/tcp, 0.0.0.0:50000->50000/tcp, 0.0.0.0:8082->8080/tcp   jenkins
```

将宿主机中的文件传输到docker容器中

```bash
docker cp jenkins.war 6f7b3143fa97:/usr/share/jenkins.war
```

若不知道是否替换成功，可是进去`docker`容器查看：

```bash
# 以root用户通过docker短ID进入容器
docker exec -it -u root 6f7b3143fa97 bash
# 进入jenkins目录
cd /usr/share
# 查看 jenkins.war 的创建时间
ls -al
```

重启docker jenkins容器

```bash
docker restart 6f7b3143fa97
```

重新进入浏览器登录就可以看到已经升级好了！

![image-20230702125850666](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021258704.png)

## jenkins插件安装失败

jenkins插件安装失败.原因有二:

+ jenkins版本太低
+ 网络问题,导致插件安装失败

解决方式有二:

+ 更新`jenkins`版本.(**参考2.1**)
+ 更换`jenkins数据源`或`手动导入jenkins插件`

### 更换`jenkins`数据源

系统管理->插件管理->Advanced settings->升级站点

![image-20230702130227565](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021302683.png)

![image-20230702130300165](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021303264.png)

滚轮划到最下面,替换`url`

```bash
# 站点
https://updates.jenkins.io/update-center.json
# 清华站点
https://jenkins-zh.gitee.io/update-center-mirror/tsinghua/update-center.json
```

之后请重新安装`jenkins 插件`

### 手动导入`jenkins`插件

首先进入官网插件界面， https://plugins.jenkins.io/

![img](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021305661.jpeg)

输入插件名称搜索，比如搜索 localization, 如果想更换简体中文语言，红框内3个插件是必须的

![img](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021306794.jpeg)

下载插件

![img](Jenkins%E4%BD%BF%E7%94%A8%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB.assets/202307021306362.jpeg)



系统管理->插件管理->Advanced settings->Deploy Plugin

将下载下来的`jenkins插件`导入

## 参考链接

- (3条消息) 更新Docker中的Jenkins版本_docker jenkins更新插件后保留_优小U的博客-CSDN博客: https://blog.csdn.net/zy1281539626/article/details/115177973