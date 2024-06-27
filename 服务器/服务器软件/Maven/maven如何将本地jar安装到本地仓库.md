# maven如何将本地jar安装到本地仓库

`jar`存放位置:   `jar: D:\express-1.0.0.jar`

举例说明, 在`pom.xml` 中信息如下

```xml
        <dependency>
            <groupId>com.sangto</groupId>
            <artifactId>express</artifactId>
            <version>1.0.0</version>
        </dependency>
```

maven安装jar到目标仓库

```bash
mvn install:install-file -Dfile=D:/express-1.0.0.jar -DgroupId=com.sangto -DartifactId=express -Dversion=1.0.0 -Dpackaging=jar
```

+ `-DgroupId`  对应 xml `gourpId`
+ `-DartifactId` 对应xml `artifactId`
+ `-Dversion`   对应 xml  `version`

+ `-Dpackaging` 对应`jar`

安装完成后,查看maven仓库下是否存在`/com/sangto/express/1.0.0/express.jar`