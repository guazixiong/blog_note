# 十三、SpringCloud2020学习笔记 Stream

## 83_Stream为什么被引入

常见MQ(消息中间件)：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka

有没有一种新的技术诞生，让我们不再关注具体MQ的细节，我们只需要用一种适配绑定的方式，自动的给我们在各种MQ内切换。（类似于Hibernate）

Cloud Stream是什么？屏蔽底层消息中间件的差异，降低切换成本，统一消息的**编程模型**

## 84_Stream是什么及Binder介绍

[官方文档1](https://spring.io/projects/spring-cloud-stream#overview)

[官方文档2](https://cloud.spring.io/spring-tloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/Spring)

[Cloud Stream中文指导手册](https://m.wang1314.com/doc/webapp/topic/20971999.html)

### 什么是Spring Cloud Stream？

官方定义**Spring Cloud Stream是一个构建消息驱动微服务的框架**。

**应用程序通过inputs或者 outputs 来与Spring Cloud Stream中binder对象交互**。

通过我们配置来binding(绑定)，而Spring Cloud Stream 的binder对象负责与消息中间件交互。所以，我们只需要搞清楚如何与Spring Cloud Stream交互就可以方便使用消息驱动的方式。

**通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。**
**Spring Cloud Stream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了`发布-订阅`、`消费组`、`分区`的三个核心概念。**

目前仅支持RabbitMQ、 Kafka.

## 85_Stream的设计思想

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251730340.png)

- **生产者/消费者之间靠消息媒介传递信息内容**
- **消息必须走特定的通道 - 消息通道 Message Channel**
- 消息通道里的消息如何被消费呢，谁负责收发处理 - **消息通道MessageChannel的子接口SubscribableChannel，由MessageHandler消息处理器所订阅**.

### **为什么用Cloud Stream？**

比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像RabbitMQ有exchange，kafka有Topic和Partitions分区

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251731529.png)

这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想**往另外一种消息队列进行迁移**，这时候无疑就是一个灾难性的，一大堆东西都要重新推倒重新做，因为它跟我们的**系统耦合**了，这时候**Spring Cloud Stream给我们提供了—种解耦合的方式。**

### Stream凭什么可以统一底层差异？

在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。**通过向应用程序暴露统一的Channel通道，使得应用程序不需要再考虑各种不同的消息中间件实现。**

**通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离**。

**Binder**(类比File输入输出流)：

- **INPUT对应于消费者**
- **OUTPUT对应于生产者**

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251732401.png)

**Stream中的消息通信方式遵循了`发布-订阅`模式**

**Topic主题进行广播**

- **在RabbitMQ就是Exchange**
- **在Kakfa中就是Topic**

## 86_Stream编码常用注解简介

### **Spring Cloud Stream标准流程套路**

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251732327.png)

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251732313.png)

- Binder - 很方便的连接中间件，屏蔽差异。

- Channel - 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。

- Source和Sink - 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。

### **编码API和常用注解**

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持RabbitMQ和Kafka                            |
| Binder          | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic,RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
| @Input          | 注解标识输入通道，通过该输乎通道接收到的消息进入应用程序     |
| @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 指信道channel和exchange绑定在一起                            |

**案例说明**

准备RabbitMQ环境（[79_Bus之RabbitMQ环境配置](https://blog.csdn.net/u011863024/article/details/114298282#)有提及）

工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801，作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802，作为消息接收模块
- cloud-stream-rabbitmq-consumer8803，作为消息接收模块

## 87_Stream消息驱动之生产者

新建Module：cloud-stream-rabbitmq-provider8801

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>LearnCloud</artifactId>
        <groupId>com.lun.springcloud</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```

YML

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址


```

主启动类StreamMQMain8801

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class,args);
    }
}

```

业务类

1.发送消息接口

```java
public interface IMessageProvider {
    public String send();
}

```

2.发送消息接口实现类

```java
import com.lun.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.MessageChannel;

import javax.annotation.Resource;
import java.util.UUID;


@EnableBinding(Source.class) //定义消息的推送管道
public class MessageProviderImpl implements IMessageProvider
{
    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send()
    {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("*****serial: "+serial);
        return null;
    }
}

```



3.Controller

```java
import com.lun.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class SendMessageController
{
    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage() {
        return messageProvider.send();
    }

}

```

测试

- 启动 7001eureka
- 启动 RabpitMq（79_Bus之RabbitMQ环境配置）
  - `rabbitmq-plugins enable rabbitmq_management` 启动rabbitmq
  - 访问 http://localhost:15672/
- 启动 8801
- 访问 - http://localhost:8801/sendMessage
  - 后台将打印serial: UUID字符串

## 88_Stream消息驱动之消费者

新建Module：cloud-stream-rabbitmq-consumer8802

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>LearnCloud</artifactId>
        <groupId>com.lun.springcloud</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```

YML

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址


```

主启动类StreamMQMain8802

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8802.class,args);
    }
}

```

业务类

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;


@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController
{
    @Value("${server.port}")
    private String serverPort;


    @StreamListener(Sink.INPUT)
    public void input(Message<String> message)
    {
        System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
    }
}

```

测试

- 启动EurekaMain7001
- 启动StreamMQMain8801
- 启动StreamMQMain8802
- 8801发送8802接收消息

## 89_Stream之消息重复消费

依照8802，克隆出来一份运行8803 - cloud-stream-rabbitmq-consumer8803。

**启动**

- RabbitMQ
- 服务注册 - 8801
- 消息生产 - 8801
- 消息消费 - 8802
- 消息消费 - 8802

**运行后有两个问题**

1. 有重复消费问题
2. 消息持久化问题

**消费**

- http://localhost:8801/sendMessage
- 目前是8802/8803同时都收到了，存在重复消费问题
- 如何解决：分组和持久化属性group（重要）

**生产实际案例**

比如在如下场景中，订单系统我们做集群部署，都会从RabbitMQ中获取订单信息，那如果一个订单同时被两个服务获取到，那么就会造成数据错误，我们得避免这种情况。这时我们就可以**使用Stream中的消息分组来解决**。

![img](%E5%8D%81%E4%B8%89%E3%80%81SpringCloud2020%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%20Stream.assets/202303251736919.png)

注意在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。不同组是可以全面消费的(重复消费)。

> Stream 采用的是发布订阅中的主题模式,默认`RoutingKey`为`#`,如果不设置`Routingkey`,发送一次消息,其他接受消息group(`queue`)也会受到相同消息

## 90_Stream之group解决消息重复消费

### 原理

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。

不同的组是可以重复消费的，同一个组内会发生竞争关系，只有其中一个可以消费。

8802/8803都变成不同组，group两个不同

group: A_Group、B_Group

8802修改YML

```yaml
spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: A_Group #<----------------------------------------关键

```

8803修改YML（与8802的类似位置 `group: B_Group`）

结论：**还是重复消费**

8802/8803实现了轮询分组，每次只有一个消费者，8801模块的发的消息只能被8802或8803其中一个接收到，这样避免了重复消费。

**8802/8803都变成相同组，group两个相同**

group: A_Group

8802修改YML`group: A_Group`

8803修改YML`group: A_Group`

结论：**同一个组的多个微服务实例，每次只会有一个拿到**

> Stream 采用的是发布订阅中的主题模式,默认`RoutingKey`为`#`,如果不设置`Routingkey`,发送一次消息,其他接受消息group(`queue`)也会受到相同消息
>
> 生产者:
>
> ```yaml
> server:
>   port: 8802
> 
> spring:
>   application:
>     name: cloud-stream-provider8802
>   cloud:
>     stream:
>       binders: # 在此处配置要绑定的rabbitmq的服务信息；
>         windrabbit: # 自定义名称，用于于binding整合
>           type: rabbit
>           environment:
>             spring:
>               rabbitmq:
>                 host: 81.70.255.49
>                 port: 5672
>                 username: admin
>                 password: admin
>       bindings:
>         saveOrderOutput:
>           destination: stream_order_exchange
>           content-type: application/json
>           binder: windrabbit
>           group: saveOrderGroup   # 设置队列名称 并不会根据group分组,发送消息时,其他分组仍会进行重复消费,需要设置分区
> #          producer:
> #            # 分区键的表达式规则
> #            # 如果`partition-key-expression` 的值是`payLoad`,将会使用所有放在GenericMessage中的数据作为分区数据,`payLoad`是消息的实体类型,可以为自定义类型入`User`,`Role`等
> #            # 如果`partition-key-expression` 的值是`headers[xxx]`,将由`MessageBuilder`类的`setHeader()`方法完成赋值
> ##            partition-key-expression: payLoad
> #            partition-count: 1
>       rabbit:
>         bindings:
>           saveOrderOutput:
>             producer:
>               # 生产者配置RabbitMq的动态路由键
>               routingKeyExpression: headers.routingKey
>   # 项目会默认连接本地mq,如果不额外添加默认配置,启动报错,但不影响正常使用
>   rabbitmq:
>     host: 81.70.255.49
>     port: 5672
>     username: admin
>     password: admin
> 
> eureka:
>   client: # 客户端进行Eureka注册的配置
>     service-url:
>       defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
> #  instance:
> #    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
> #    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
> #    instance-id: send-8801.com  # 在信息列表时显示主机名称
> #    prefer-ip-address: true     # 访问的路径变为IP地址
> 
> 
> ```
>
> ```java
>     /**
>      * 消息发送 + 自定义动态路由
>      *
>      * @param msg
>      */
>     public void sendMsg(String msg) {
>         // 消息发送 + 自定义动态路由
>         String routingKey = "saveOrderInput";
>         channel.send(MessageBuilder.withPayload(msg).setHeader("routingKey",routingKey).build());
>         log.info("消息发送成功：" + msg);
>     }
> ```
>
> 消费者:
>
> ```yaml
> server:
>   port: 8805
> 
> spring:
>   application:
>     name: cloud-stream-consumer
>   cloud:
>     stream:
>       # topic 主题订阅模式
>       binders: # 在此处配置要绑定的rabbitmq的服务信息；
>         windrabbit: # 自定义名称，用于于binding整合
>           type: rabbit
>           environment:
>             spring:
>               rabbitmq:
>                 host: 81.70.255.49
>                 port: 5672
>                 username: admin
>                 password: admin
>       bindings: # 服务的整合处理
>         saveOrderInput: #这个是消息通道的名称 ---> 保存订单输入通道
>           destination: stream_order_exchange     #exchange名称,交换模式默认是topic;把SpringCloud stream的消息输出通道绑定到RabbitMQ的exchange-saveOrder交换器。
>           content-type: application/json      #设置消息的类型，本次为json
>           binder: windrabbit
>           group: saveOrderGroup               #分组
> #          consumer:
> #            partitioned: true # 支持分区
> #            instance-index: 0 # 当前消费者索引
> #            instance-count: 1 # 消费者总数
>         orderInput: #这个是消息通道的名称 ---> 订单输入通道
>           destination: stream_order_exchange     #exchange名称,交换模式默认是topic;把SpringCloud stream的消息输出通道绑定到RabbitMQ的exchange-saveOrder交换器。
>           content-type: application/json      #设置消息的类型，本次为json
>           binder: windrabbit
>           group: orderGroup2               #分组
> #          consumer:
> #            partitioned: true # 支持分区
> #            instance-index: 0 # 当前消费者索引1
> #            instance-count: 2 # 消费者总数
>         orderOutput: #这个是消息通道的名称 ---> 订单输出通道
>           destination: stream_order_exchange     #exchange名称,交换模式默认是topic;把SpringCloud stream的消息输出通道绑定到RabbitMQ的exchange-saveOrder交换器。
>           content-type: application/json      #设置消息的类型，本次为json
>           binder: windrabbit
>           group: orderGroup2            #分组
> #          consumer:
> #            partitioned: true # 支持分区
> #            instance-index: 1 # 当前消费者索引2
> #            instance-count: 2 # 消费者总数
>       # 这一部分是给上边声明的bindings添加配置的，例如队列的ttl，还有要不要给队列配置死信队列
>       rabbit:
>         bindings:
>           saveOrderInput:  # 对saveOrderInput设置相关设置
>             consumer:
>               #ttl: 20000 # 默认不做限制，即无限。消息在队列中最大的存活时间。当消息滞留超过ttl时，会被当成消费失败消息，即会被转发到死信队列或丢弃.即消息在队列中存活的最大时间为 20s
>               # DLQ相关
>               autoBindDlq: true # 是否自动声明死信队列（DLQ）并将其绑定到死信交换机（DLX）。默认是false。
>               republishToDlq: true
>               #deadLetterExchange: exchange-order-dlq  #绑定exchange
>               #deadLetterQueueName: exchange-order-dlq.saveOrderInput  #死信队列名字：exchanName.queueName
>               binding-routing-key: saveOrderInput
>           orderInput: # 对saveOrderInput设置相关设置
>             consumer:
>               binding-routing-key: orderInput
>   # 项目会默认连接本地mq,如果不额外添加默认配置,启动报错,但不影响正常使用
>   rabbitmq:
>     host: 81.70.255.49
>     port: 5672
>     username: admin
>     password: admin
> 
> eureka:
>   client: # 客户端进行Eureka注册的配置
>     service-url:
>       defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
> #  instance:
> #    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
> #    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
> #    instance-id: send-8801.com  # 在信息列表时显示主机名称
> #    prefer-ip-address: true     # 访问的路径变为IP地址
> ```

## 91_Stream之消息持久化

通过上述，解决了重复消费问题，再看看持久化。

停止8802/8803并去除掉8802的分组group: A_Group，8803的分组group: A_Group没有去掉。

8801先发送4条消息到RabbitMq。

先启动8802，**无分组属性配置**，**后台没有打出来消息**。

再启动8803，**有分组属性配置**，**后台打出来了MQ上的消息。(消息持久化体现)**