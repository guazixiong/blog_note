# Linux服务器安装maven

+ maven 官网下载安装包
+ 安装
+ 配置环境变量

在 linux 上安装 maven，步骤如下

笔者这里使用 centos7，apache-maven-3.8.5

## 1、maven 官网下载安装包

maven 官网地址：https://maven.apache.org/download.cgi

下载完成后，上传到 linux

![img](Linux%E5%AE%89%E8%A3%85maven.assets/202305302251704.png)

## 2、安装 maven

创建 maven 文件夹

```bash
mkdir -p /usr/local/maven
```

解压 maven 安装包到 maven 文件夹

```bash
tar -zxvf apache-maven-3.8.5-bin.tar.gz -C /usr/local/maven
```

进入maven 目录下的conf目录

```bash
cd /usr/local/maven/apache-maven-3.8.5/conf
```

创建 maven 资源库目录

```
mkdir -p /m2/repository
```

编辑 settings.xml 文件,**将原文件内容全部删除，添加新的配置内容**

```bash
:1,.d
```

修改了资源库位置，添加了阿里云国内镜像

```bash
<?xml version="1.0" encoding="UTF-8"?>
 
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
 
  <localRepository>/m2/repository</localRepository>
  
  <pluginGroups>
  </pluginGroups>
 
  <proxies>    
  </proxies>
 
  <servers>
  </servers>
 
  <mirrors>
    <mirror>  
   	  <id>alimaven</id>  
   	  <name>aliyun maven</name>  
	  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
   	  <mirrorOf>central</mirrorOf>          
    </mirror> 
  </mirrors>
 
  <profiles>
  </profiles>
</settings>
```

```bash
:wq
```

## 3、添加环境变量

```bash
vi /etc/profile
```

添加 maven 环境变量内容

```bash
MAVEN_HOME=/usr/local/maven/apache-maven-3.8.5
PATH=$MAVEN_HOME/bin:$PATH
export MAVEN_HOME PATH
```

```bash
:wq
```

重新加载配置文件

```bash
source /etc/profile
```

测试

```bash
mvn -version
```

































