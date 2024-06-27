# 	Docker 搭建Jenkins部署maven项目

## 环境准备

本文采用`docker`安装`jenkins`,通过创建容器的方式,去构建maven项目,笔者在这里放一下自己的环境

| 环境     | 版本               | 备注 |
| -------- | ------------------ | ---- |
| 操作系统 | centos 7.0         |      |
| jdk      | jdk1.8.0_361       |      |
| maven    | apache-maven-3.9.2 |      |
| git      | git version 2.20.0 |      |
| docker   | 24.0.2             |      |
| jenkins  | 2.4.12             |      |

## 下载Jenkins

```bash
#查询镜像
docker search jenkins
```

![image-20230723102415723](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231024777.png)

这里使用的是第二个镜像(因为第一个镜像的docker版本较低) (后期我们也是大概率是要去更新jenkins版本的,因为docker中jenkins版本远落后于官方jenkins版本,具体更新看一下文章)

## 启动Jenkins容器

```bash
#创建文件夹
mkdir -p /home/jenkins_home
#权限
chmod 777 /home/jenkins_home
```

```bash
docker run -d -u root -p 8082:8080 -p 8100:8100 -p 50000:50000 \
  --name myJenkins \
  -v /home/jenkins_home:/var/jenkins_home \
  -v /usr/local/java/jdk1.8.0_361:/usr/local/java/jdk1.8.0_361 \
  -v /usr/local/maven/apache-maven-3.9.2:/usr/local/maven/apache-maven-3.9.2 \
  -v /usr/local/git/git-2.20.0:/usr/local/git/git-2.20.0 \
  jenkins/jenkins
```

> **注意: 我使用的是root用户进行访问,为了防止后续进入容器内部权限不足导致jenkins构建项目提示权限不足**
>
> 对外挂载目录:
>
> + 8082: 替代jenkins访问目录8080
> + 8100: maven项目对外暴露端口
> + 50000:  Jenkins在内部使用此端口，以允许构建从执行程序连接到主Jenkins服务器。
>
> 挂载本地环境到jenkins中
>
> + 服务器jenkins目录,方便后续更新镜像后继续使用原来的工作目录
> + jdk环境:jenkins默认jdk11,因此构建jdk8项目会存在问题
> + maven环境:jenkins并不提供maven配置,除非通过界面手动安装
> + git环境,我们需要从远程代码库中获取实时更新代码,并自动构建,必不可少(或者在jenkins页面手动下载)

```bash
#日志查看,会提供初始密码,不查看也可以
docker logs jenkins
```

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231030615.png)

在浏览器中输入:http://serverIp:port/访问jenkins

+ serverIp为docker宿主机的ip，
+ port即为宿主机映射的端口。

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231031909.png)

这里要求输入初始的管理员密码，根据提示密码在`/var/jenkins_home/secrets/initialAdminPassword`这个文件中，注意这个路径是Docker容器中的，所以我们通过如下命令获取一下

```bash
# 方式一: 在jenkins容器内获取
docker exec 容器id cat /var/jenkins_home/secrets/initialAdminPassword

# 方式二: 在本地数据卷中获取
cat /home/jenkins_home/secrets/initialAdminPassword
```

## 安装插件

> 解锁jenkins后,进入安装插件这一步,由于jenkins插件和jenkins的版本有关系,因此有些插件可能会安装失败,所以为了避免插件安装失败,笔者建议,先将jenkins升级为最新版本(**具体替换看Jenkins问题汇总**)
>
> > 网上说的替换插件源,是由于jenkins插件需要科学上网,只能提高下载速度,但无法解决jenkins版本落后导致的插件安装问题
> >
> > 附: 清华大学官方镜像数据源
> >
> > ```bash
> > https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
> > ```

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231033625.png)

选择推荐插件安装

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231046237.png)

## 创建用户

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231047392.png)

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231047686.png)

## 配置环境

### 应用环境

> **所有的环境命令均指的是docker容器内jenkin的环境,不要傻傻的写linux服务器的环境配置**
>
> 由于笔者在最早创建docker容器时,已经将服务器环境与jenkins容器内部做了映射,虽然路径一致,但一定记住,是jenkins容器内部环境路径

管理->Global Tool Configuration

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231050131.png)

配置JDK

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231051269.png)

> 注意：**这里配置的是jekins的jdk，由于jekins挂载目录和宿主机目录相同，所以别以为是宿主机目录，一定是jekins的jdk目录，jekins的jdk目录映射到宿主机目录**
> /usr/local/java/jdk1.8.0_361

配置maven

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231054940.png)

> 注意事项同上

### 集成码云(Gitee)

> 笔者这里使用的是gitee作为远程代码存储库,因此这里配置gitee

配置码云(Gitee)前,先在jenkins内部下载gitee插件

管理->Manage Plugins

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231056762.png)

下载完成后,配置git权限,保证jenkins容器内部可以访问到git项目

#### 方式一: 配置私人令牌

码云用户设置处,生成私人令牌

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231058100.png)

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231058341.png)

在管理-> 凭据配置 -> 凭据 -> 用户 -> 全局凭据里,类型选择gitee api令牌

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231058418.png)

#### 配置SSH公钥

> 这里通过生成ssh公钥来获取访问gitee的权限,注意:
>
> + **ssh公钥要求使用的docker容器内部jenkins生成的ssh公钥**
>
> **直接使用服务器的ssh公钥,在jenkins构建项目时,会提示没有git权限**

## 创建maven项目

> 这里需要我们包含有maven插件
>
> + 如果是按照推荐插件来安装的,请跳过
> + 选择安装的,请安装maven插件
>
> ![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231103086.png)

创建项目

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231103959.png)

配置构建项目时,需要的环境

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231105567.png)

![img](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231105490.png)

> 选中我们提前配置好的git凭据,这里如果我们的凭据配置成功,仓库地址下并不会出现git访问请求失败的错误提示

继续设置我们的其他步骤

我们如果想要实现推送git请求后,jenkins立马获取最新代码并重新构建maven项目,可以**勾选Gitee webhook 触发构建，需要在 Gitee webhook 中填写 URL:** 

![image-20230723172044724](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231720806.png)

构建完成之后,执行自定义shell脚本,用来启动maven项目

```shell
BUILD_ID=dontKillMe
#代码地址
baseDir=/home/jenkins_home/workspace/YunWeiFang
# jar名称
jarName=yunweifang.jar
# jar地址
jarDir=/home/jenkins_home/workspace/YunWeiFang/yunweifang/target
#日志
logDir=YunWeiFangLog.out
# 关闭进程
tpid=`jps | grep "yunweifang.jar" | awk '{ print $1 }'`
if [ ${tpid} ]; then
        kill -9 $tpid
fi
echo "server is stop"
cd $baseDir
mvn clean install package
echo "mvn package over"
cd $jarDir
nohup java -jar $jarName > $logDir
```

保存配置信息,点击左侧菜单栏,立即构建项目

![image-20230723172350323](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231723376.png)

![image-20230723172529486](Jenkins%E9%83%A8%E7%BD%B2maven%E9%A1%B9%E7%9B%AE.assets/202307231725541.png)