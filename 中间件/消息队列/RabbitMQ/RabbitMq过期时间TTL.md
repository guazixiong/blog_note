# RabbitMq过期时间TTL

> 相关代码 https://gitee.com/guazixiong/study-computer.git    **springboot-rabbitmq-test**

## TTL

### 1、概述

过期时间TTL表示可以对消息设置预期的时间，在这个时间内都可以被消费者接收获取；过了之后消息将自动被删除。RabbitMQ可以对**消息和队列**设置TTL。目前有两种方法可以设置。

- **第一种方法是通过队列属性设置，队列中所有消息都有相同的过期时间。**
- **第二种方法是对消息进行单独设置，每条消息TTL可以不同。**

**如果上述两种方法同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准**。消息在队列的生存时间一旦超过设置的TTL值，就**称为dead message被投递到死信队列**， 消费者将无法再收到该消息。

```java
@Configuration
public class TtlRabbitConfig {

    /**
     * 过期队列.
     *
     * @return 新建过期队列
     */
    @Bean
    public Queue ttlQueue() {
        // 设置过期时间
        Map<String,Object> args = new HashMap<>();
        args.put("x-message-ttl", 5000);
        // 绑定死信队列
        args.put("x-dead-letter-exchange","dead_order_exchange");
        args.put("x-dead-letter-routing-key","dead");
        return new Queue("ttl.test.queue", true,false,false,args);
    }

    /**
     * 正常队列,消息设置过期时间.
     *
     * @return 新建过期队列
     */
    @Bean
    public Queue ttlMessageQueue() {
        return new Queue("ttl.message.queue", true,false,false);
    }

    @Bean
    public DirectExchange ttlOrderExchange() {
        return new DirectExchange("ttl_order_exchange", true, false);
    }

    @Bean
    public Binding ttlQueueBind() {
        return BindingBuilder.bind(ttlQueue()).to(ttlOrderExchange()).with("ttl");
    }

    @Bean
    public Binding ttlMessageBind() {
        return BindingBuilder.bind(ttlMessageQueue()).to(ttlOrderExchange()).with("ttlMessage");
    }
}

```

```java
    /**
     * ttl 消息过期删除.
     *
     * @param userId 用户id
     * @param productId 商品id
     * @param num 商品数量
     */
    public void makeOrderTtlMessage(Long userId, Long productId, int num) {
        String orderNumber = UUID.randomUUID().toString();
        System.out.println("用户 " + userId + ",订单编号是：" + orderNumber);
        rabbitTemplate.convertAndSend(ttlExchangeName, "ttlMessage", orderNumber, new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                // 取过期队列与过期消息的时间最小值,String字符串
                message.getMessageProperties().setExpiration("6000");
                message.getMessageProperties().setContentEncoding("UTF-8");
                return message;
            }
        });
    }
```

参数` x-message-ttl `的值 必须是非负 32 位整数 (0 <= n <= 2^32-1) ，以毫秒为单位表示 TTL 的值。这样，值 6000 表示存在于 队列 中的当前 消息 将最多只存活 6 秒钟。

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141112188.png)

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141112471.png)

> expiration 字段以微秒为单位表示 TTL 值。且与 x-message-ttl 具有相同的约束条件。因为 expiration 字段必须为字符串类型，broker 将只会接受以字符串形式表达的数字。
> 当同时指定了 queue 和 message 的 TTL 值，则两者中较小的那个才会起作用。

## 死信队列

`DLX`，全称为`Dead-Letter-Exchange` , 可以称之为**死信交换机**，也有人称之为死信邮箱。**当消息在一个队列中变成死信(dead message)之后，它能被重新发送到另一个交换机中，这个交换机就是DLX ，绑定DLX的队列就称之为死信队列。**
消息变成死信，可能是由于以下的原因：

- **消息被拒绝**
- **消息过期**
- **队列达到最大长度**

DLX也是一个正常的交换机，和一般的交换机没有区别，它能在任何的队列上被指定，实际上就是设置某一个队列的属性。**当这个队列中存在死信时，Rabbitmq就会自动地将这个消息重新发布到设置的DLX上去，进而被路由到另一个队列，即死信队列**。
要想使用死信队列，只需要在定义队列的时候设置队列参数 `x-dead-letter-exchange` 指定交换机即可。

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141113385.png)

未过期：

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141113416.png)

过期后：

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141114402.png)

![](RabbitMq%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4TTL.assets/202303141114942.png)