# RabbitMQ的核心组成部分

## 01、RabbitMQ的核心组成部分

![img](RabbitMQ%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.assets/202303021518946.png)

核心概念：

- **Server**：又称Breoker ,接受客户端的连接，实现AMQP实体服务。 安装rabbitmq-server
- **Connection**：连接，应用程序与Breoker的网络连接 TCP/IP/ 三次握手和四次挥手
- **Channel**：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道，客户端可以建立对各Channel，每个Channel代表一个会话任务。
- **Message** :消息：服务与应用程序之间传送的数据，由Properties和body组成，Properties可是对消息进行修饰，比如消息的优先级，延迟等高级特性，Body则就是消息体的内容。
- **Virtual Host** 虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个虚拟主机理由可以有若干个Exhange和Queueu，同一个虚拟主机里面不能有相同名字的Exchange
- **Exchange**：交换机，接受消息，根据路由键发送消息到绑定的队列。(==不具备消息存储的能力==)
- **Bindings**：Exchange和Queue之间的虚拟连接，binding中可以保护多个routing key.
- **Routing key**：是一个路由规则，虚拟机可以用它来确定如何路由一个特定消息(筛选特定的Consumer)。
- **Queue**：队列：也成为Message Queue,消息队列，保存消息并将它们转发给消费者。

## 02、RabbitMQ整体架构是什么样子的？

![img](RabbitMQ%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.assets/202303021518493.png)

## 03、RabbitMQ的运行流程

生产者绑定交换机Exchange和指定RoutingKey,将消息发送至Breoker节点中,消费者Consumer订阅Breoker,通过唯一的RoutingKey从交换机中Exchange中获取队列信息,再进行业务处理.

![img](RabbitMQ%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.assets/202303021518540.png)

## 03、RabbitMQ支持消息的模式

> RabbitMQ-15、简单模式理解 | KuangStudy | 飞哥 | 狂神说 | 学相伴_哔哩哔哩_bilibili
> https://www.bilibili.com/video/BV1dX4y1V73G/?p=15&spm_id_from=pageDriver&vd_source=88fd64ae1cd8df6192cc3b47b05f65b1
>
> 官网：https://www.rabbitmq.com/getstarted.html

### 03-1、简单模式 Simple

- 参考第12章节

一个生产者，一个消费者，不需要设置交换机，使用默认交换机；

---

不涉及Exchange和RoutingKey

---

### 03-2、工作模式 Work

- web操作查看视频
- 类型：无
- 特点：分发机制

一个生产者，多个消费者（竞争关系），不需要设置交换机（使用默认交换机）；

#### 轮询模式 一个消费者一条，按均分配 

```java
public class Work2 {
    public static void main(String[] args) {
        // 1: 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 设置连接属性
        connectionFactory.setHost("47.104.141.27");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        Connection connection = null;
        Channel channel = null;
        try {
            // 3: 从连接工厂中获取连接
            connection = connectionFactory.newConnection("消费者-Work2");
            // 4: 从连接中获取通道channel
            channel = connection.createChannel();
            // 5: 申明队列queue存储消息
            // 6： 定义接受消息的回调
            Channel finalChannel = channel;
            finalChannel.basicQos(1);
            // 轮询分发autoAck为true,
            finalChannel.basicConsume("queue1", true, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery delivery) throws IOException {
                    try{
                        System.out.println("Work2-收到消息是：" + new String(delivery.getBody(), "UTF-8"));
                        Thread.sleep(200);
                    }catch(Exception ex){
                        ex.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                }
            });
            System.out.println("Work2-开始接受消息");
            System.in.read();
        } catch (Exception ex) {
            ex.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 7: 释放连接关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }
    }
}
```

#### 公平分发 按劳分配

```java
public class Work2 {
    public static void main(String[] args) {
        // 1: 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 设置连接属性
        connectionFactory.setHost("47.104.141.27");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        Connection connection = null;
        Channel channel = null;
        try {
            // 3: 从连接工厂中获取连接
            connection = connectionFactory.newConnection("消费者-Work2");
            // 4: 从连接中获取通道channel
            channel = connection.createChannel();
            // 5: 申明队列queue存储消息
            // 同一时刻，服务器只会推送一条消息给消费者
            //channel.basicQos(1);
            // 6： 定义接受消息的回调
            Channel finalChannel = channel;
            finalChannel.basicQos(1);
             // 公平分发autoAck为fasle,
            finalChannel.basicConsume("queue1", false, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery delivery) throws IOException {
                    try{
                        System.out.println("Work2-收到消息是：" + new String(delivery.getBody(), "UTF-8"));
                        Thread.sleep(200);
                        // 公平分发
                        finalChannel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
                    }catch(Exception ex){
                        ex.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                }
            });
            System.out.println("Work2-开始接受消息");
            System.in.read();
        } catch (Exception ex) {
            ex.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 7: 释放连接关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }
    }
}
```

### 03-3、发布订阅模式

- web操作查看视频
- 类型：fanout
- 特点：Fanout—发布与订阅模式，是一种广播机制，它是没有路由key的模式。

需要设置类型为 fanout-广播的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消息发送到绑定的队列；

---

涉及Exchange,不涉及RoutingKey

---

### 03-4、路由模式

- web操作查看视频
- 类型：direct
- 特点：有routing-key的匹配模式

需要设置类型为 direct的交换机， 交换机和队列进行绑定，并且指定routing key，当发送消息到交换机后，交换机会根据routing key 将消息发送到对应队列；

---

涉及Exchange,RoutingKey

---

### 03-5、主题Topic模式

- web操作查看视频
- 类型：topic
- 特点：模糊的routing-key的匹配模式

需要设置类型为 topic的交换机， 交换机和队列进行绑定， 并且指定通配符方式的routing key， 当发送消息到交换机后，交换机会根据 routing key将消息发送到对应的队列；

---

Topic 类型与 Direct 相比，都是可以根据 RoutingKey 把消息路由到不同的队列。

只不过 **Topic 类型Exchange 可以让队列在绑定 Routing key 的时候使用通配符！**

Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert

通配符规则：

- `#` 匹配0个,或多个词
- `*` 匹配不多不少恰好1个词

**例如：item.# 能够匹配 item.insert.abc 或者 item.insert，item.* 只能匹配 item.inser**

---

### 03-6、参数模式

- web操作查看视频
- 类型：headers
- 特点：参数匹配模式

## 小结

- rabbitmq发送消息一定有一个交换机

- 如果队列没有指定交换机会默认绑定一个交换机

  ![img](RabbitMQ%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.assets/202303021518203.png)

![img](RabbitMQ%E7%9A%84%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.assets/202303021518382.png)