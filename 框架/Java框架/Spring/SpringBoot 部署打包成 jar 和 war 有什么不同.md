# SpringBoot 部署打包成 jar 和 war 有什么不同?

首先给大家来讲一个我们遇到的一个奇怪的问题:

1. **我的一个springboot项目，用mvn install打包成jar，换一台有jdk的机器就直接可以用java -jar 项目名.jar的方式运行，没任何问题，为什么这里不需要tomcat也可以运行了？**
2. **然后我打包成war放进tomcat运行，发现端口号变成tomcat默认的8080（我在server.port中设置端口8090）项目名称也必须加上了。**

**也就是说我在原来的机器的IDEA中运行，项目接口地址为 ip:8090/listall,打包放进另一台机器的tomcat就变成了ip:8080/项目名/listall**。这又是为什么呢？

- 通过jar运行实际上是启动了内置的tomcat,所以用的是应用的配置文件中的端口
- 直接部署到tomcat之后，内置的tomcat就不会启用，所以相关配置就以安装的tomcat为准，与应用的配置文件就没有关系了

哎，现在学编程的基本都不会教历史了，也没人有兴趣去钻研。

总体来说吧，很多年前，Sun 还在世的那个年代，在度过了早期用 C++写 Html 解析器的蛮荒时期后，有一批最早的脚本程序进入了 cgi 时代，此时的 Sun 决定进军这个领域，为了以示区别并显得自己高大上，于是研发了 servlet 标准，搞出了最早的 jsp。并给自己起了个高大上的称号 JavaEE （ Java 企业级应用标准，其实不就是一堆服务器以 http 提供服务吗，吹逼）。

既然是企业级标准那自然得有自己的服务器标准。于是 Servlet 标准诞生，以此标准实现的服务器称为 Servle 容器服务器，Tomcat 就是其中代表，被 Sun 捐献给了 Apache 基金会，那个时候的 Web 服务器还是个高大上的概念，当时的 Java Web 程序的标准就是 War 包(其实就是个 Zip 包)，这就是 War 包的由来。

后来随着服务器领域的屡次进化，人们发现我们为什么要这么笨重的 Web 服务器，还要实现一大堆 Servlet 之外的管理功能，简化一下抽出核心概念 servlet 不是更好吗，最早这么干的似乎是 Jetty，出现了可以内嵌的 Servelet 服务器。

去掉了一大堆非核心功能。后来 tomcat 也跟进了，再后来，本来很笨重的传统 JavaEE 服务器 Jboss 也搞了个 undertow 来凑热闹。正好这个时候微服务的概念兴起，“ use Jar，not War ”。要求淘汰传统 Servlet 服务器的呼声就起来了。

## jar包和war包的区别

1、war是一个web模块，其中需要包括WEB-INF，是可以直接运行的WEB模块；jar一般只是包括一些class文件，在声明了Main_class之后是可以用java命令运行的。

2、war包是做好一个web应用后，通常是网站，打成包部署到容器中；jar包通常是开发时要引用通用类，打成包便于存放管理。

3、war是Sun提出的一种Web应用程序格式，也是许多文件的一个压缩包。这个包中的文件按一定目录结构来组织；classes目录下则包含编译好的Servlet类和Jsp或Servlet所依赖的其它类（如JavaBean）可以打包成jar放到WEB-INF下的lib目录下。

JAR文件格式以流行的ZIP文件格式为基础。与ZIP文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。

### 【格式特点】：

- **安全性**　可以对 JAR 文件内容加上数字化签名。这样，能够识别签名的工具就可以有选择地为您授予软件安全特权，这是其他文件做不到的，它还可以检测代码是否被篡改过。
- **减少下载时间**　如果一个 applet 捆绑到一个 JAR 文件中，那么浏览器就可以在一个 HTTP 事务中下载这个 applet 的类文件和相关的资源，而不是对每一个文件打开一个新连接。
- **压缩** JAR 格式允许您压缩文件以提高存储效率。
- **传输平台扩展** Java 扩展框架（Java Extensions Framework）提供了向 Java 核心平台添加功能的方法，这些扩展是用 JAR 文件打包的（Java 3D 和 JavaMail 就是由 Sun 开发的扩展例子）。

> WAR文件就是一个Web应用程序，建立WAR文件，就是把整个Web应用程序（不包括Web应用程序层次结构的根目录）压缩起来，指定一个war扩展名。

### 【建立的条件】：

- 需要建立正确的Web应用程序的目录层次结构。
- 建立`WEB-INF`子目录，并在该目录下建立classes与lib两个子目录。
- 将Servlet类文件放到`WEB-INF\classes`目录下，将Web应用程序所使用Java类库文件（即JAR文件）放到`WEB-INF\lib`目录下。
- 将JSP页面或静态HTML页面放到上下文根路径下或其子目录下。
- 建立`META-INF`目录，并在该目录下建立`context.xml`文件。

**下面给大家讲讲怎么将springboot项目打包成jar和war**

SpringBoot项目打包成jar很简单，也是SpringBoot的常用打包格式；本篇博客将SpringBoot打包成jar和war两种方式都记录下来；

先介绍将SpringBoot打包成jar包的方式：（以下示例是在idea中演示）

### 一、打包成jar

1）先new 一个`Spring Starter Project`

![image-20231120174601047](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174601047.png)

这里注意packaging默认为jar,不用修改.

2）创建完成后项目的pom如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 <modelVersion>4.0.0</modelVersion>
 <parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.4.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
 </parent>
 <groupId>com.example</groupId>
 <artifactId>demo</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <name>demo</name>
 <description>Demo project for Spring Boot</description>
 
 <properties>
  <java.version>1.8</java.version>
 </properties>
 
 <dependencies>
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter</artifactId>
  </dependency>
 
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
  </dependency>
 </dependencies>
 
 <build>
  <plugins>
   <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
   </plugin>
  </plugins>
 </build>
 
</project>
```

3）打成jar包（通过maven命令的方式）：

在Terminal窗口，使用 `mvn clean package` 命令打包：

![image-20231120174617031](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174617031.png)

然后在target目录下就能看到打包好的jar包了

![image-20231120174624640](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174624640.png)

### 二、打包成war包形式

1）可以在刚才创建的项目上做改动,首先打包成war需要一个`ServletInitializer`类，这个类的位置需要和启动类在同一个文件下

![image-20231120174633993](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174633993.png)

> 如果一开始选择war包形式，会自动创建此类

2）修改`pom.xml`

修改`pom.xml`的war将原先的jar改为war;

3)如果我们的SpringBoot是使用html作为前端页面开发没有问题，但是如果我们想用jsp开发，这个时候就需要配置一些依赖了：主要是排除SpringBoot的内置Tomcat，添加`javax.servlet-api`和`tomcat-servlet-api`（SpringMVC还需要配置后缀）；

最后的`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 <modelVersion>4.0.0</modelVersion>
 <parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.4.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
 </parent>
 <groupId>com.example</groupId>
 <artifactId>demo</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <packaging>war</packaging>
 <name>demo</name>
 <description>Demo project for Spring Boot</description>
 
 <properties>
  <java.version>1.8</java.version>
 </properties>
 
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
 
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-servlet-api</artifactId>
            <version>8.0.36</version>
            <scope>provided</scope>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
 
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
  </dependency>
 </dependencies>
 
 <build>
  <plugins>
   <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
   </plugin>
  </plugins>
 </build>
 
</project>
```

因为SpringBoot默认推荐的是html，而不是jsp；经过上面的修改就可以使用jsp进行开发了；

4）打包成war：使用`mvn clean package`

如下：

![image-20231120174644316](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174644316.png)

打包成功后，就可以将war包放在tomcat下的webapps下，然后运行tomcat，启动项目了；

记录下来，以后用到的时候看 ^_^；

> 当然了，在创建项目的时候直接选择package为war,直接就能打成war包了

当选择war为打包方式创建项目时，`ServletInitializer`是默认直接创建的

![image-20231120174653499](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174653499.png)

此时，pom文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 <modelVersion>4.0.0</modelVersion>
 <parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.4.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
 </parent>
 <groupId>com.example</groupId>
 <artifactId>demo</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <packaging>war</packaging>
 <name>demo</name>
 <description>Demo project for Spring Boot</description>
 
 <properties>
  <java.version>1.8</java.version>
 </properties>
 
 <dependencies>
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
 
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
  </dependency>
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
  </dependency>
 </dependencies>
 
 <build>
  <plugins>
   <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
   </plugin>
  </plugins>
 </build>
 
</project>
```

直接`mvn clean package`就能打包成功

![image-20231120174717584](SpringBoot%20%E9%83%A8%E7%BD%B2%E6%89%93%E5%8C%85%E6%88%90%20jar%20%E5%92%8C%20war%20%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C.assets/image-20231120174717584.png)

> *感谢阅读，希望对你有所帮助 :)* 
>
> *来源：https://blog.csdn.net/weixin_40910372/*