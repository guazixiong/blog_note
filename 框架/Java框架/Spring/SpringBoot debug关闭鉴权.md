## SpringBoot DEBUG 关闭鉴权

### pom.xml文件中添加坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Application启动类注解修改

```java
原注解:
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
修改后:@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class,org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class})
```

### 访问项目接口

![image-20231120173718931](SpringBoot%20debug%E5%85%B3%E9%97%AD%E9%89%B4%E6%9D%83.assets/image-20231120173718931.png)

