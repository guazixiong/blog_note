# 十四、SpringCloud2020学习笔记 Sleuth

## 92_Sleuth是什么

为什么会出现这个技术？要解决哪些问题？

在微服务框架中，**一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。**



![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251744156.png)

服务节点互相调用

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251744434.png)

**是什么**

- https://github.com/spring-cloud/spring-cloud-sleuth
- **Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案**
- **在分布式系统中提供追踪解决方案并且兼容支持了zipkin**

**解决**

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251745849.png)

## 93_Sleuth之zipkin搭建安装

### zipkin

下载

- SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可
- `https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/`
- `zipkin-server-2.12.9-exec.jar`

**运行jar**

```bash
java -jar zipkin-server-2.12.9-exec.jar
```

**运行控制台**  `http://localhost:9411/zipkin/`

### **术语**

**完整的调用链路,表示一请求链路，一条链路通过`Trace ld`唯一标识，`Span标识`发起的请求信息，各span通过`parent id`关联起来**

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251747433.png)

—条链路通过Trace ld唯一标识，Span标识发起的请求信息，各span通过parent id关联起来。

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251747632.png)

整个链路的依赖关系如下：

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251747303.png)

名词解释

- Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识
- span：表示调用链路来源，通俗的理解span就是一次请求信息

## 94_Sleuth链路监控展现

2.服务提供者

cloud-provider-payment8001

POM

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>

```

YML

```yaml

spring:
  application:
    name: cloud-payment-service

  zipkin: #<-------------------------------------关键 
      base-url: http://localhost:9411
  sleuth: #<-------------------------------------关键
    sampler:
    #采样率值介于 0 到 1 之间，1 则表示全部采集
    probability: 1
    
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456

```

业务类PaymentController

```java
@RestController
@Slf4j
public class PaymentController {
    
    ...
    
 	@GetMapping("/payment/zipkin")
    public String paymentZipkin() {
        return "hi ,i'am paymentzipkin server fall back，welcome to here, O(∩_∩)O哈哈~";
    }    
}

```

3.服务消费者(调用方)

cloue-consumer-order80

POM

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>

```

```yaml
spring:
    application:
        name: cloud-order-service
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
        probability: 1

```

```java
    // ====================> zipkin+sleuth
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin()
    {
        String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
        return result;
    }
}

```

4.依次启动eureka7001/8001/80 - 80调用8001几次测试下

5.打开浏览器访问: http://localhost:9411

![img](%E5%8D%81%E5%9B%9B%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Sleuth.assets/202303251749007.png)