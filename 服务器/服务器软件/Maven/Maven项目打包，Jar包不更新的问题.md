# Maven项目打包，Jar包不更新的问题

## 问题描述

有公共包`common`和依赖公共包的`demo`,由于修改了`common`包的`domain`实体类,然后继续打包`demo`,生成的`jar`中并不包含修改的实体类代码.

![image-20230702123058024](Maven%E9%A1%B9%E7%9B%AE%E6%89%93%E5%8C%85%EF%BC%8CJar%E5%8C%85%E4%B8%8D%E6%9B%B4%E6%96%B0%E7%9A%84%E9%97%AE%E9%A2%98.assets/202307021230086.png)

## 问题原因

由于我是先`mvn clean pakcage` 项目`demo`,后打包的`common`包,所以导致在打包`demo`时,maven仓库的依赖仍是修改代码前的`common`,因此问题也就很清楚了,重新打包`common`包,然后打包引用依赖`demo`包.

原:

* 先打包`demo`
* 后打包`common`

现:

* 先打包`common`
* 后打包`demo`

归根到底,还是个人对于maven的打包特性不了解,

以`mvn clean install package` 为例

+ `clean` 操作会清除当前项目下的`target`目录,清空历史打包记录
+ `install`操作是将代码代码更新到本地的maven仓库中,便于其他模块直接引入`jar`和解析jar
+ `package`操作则是将当前项目下生成`target`目录,并将项目打成`jar`,便于我们去使用

> 而本文的问题,则是,当项目`demo`去打包时,他会主动去获取maven仓库中的`common`包,但此时`common`包未经过`mvn install`更新操作,导致获取的还是原始代码,自然会报错.