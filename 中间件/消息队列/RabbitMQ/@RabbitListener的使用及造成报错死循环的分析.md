# 【RabbitMQ】@RabbitListener的使用及造成报错死循环的分析

## 前置知识：

### 一、 @RabbitListener的使用

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

#### 1.1 作用在方法上

> 指定队列名，多个方法使用@RabbitListener，可以绑定多个消费实现

指定队列的消费方法
普通用法

```java
@RabbitListener(queues = "queue.order")
public void handleMsg(String msg) {
    
}
```

#### 1.2 作用在类上

> 指定队列名，在类上配置@RabbitListener 指定队列名 。
> **@RabbitHandler绑定在类中的多个方法上，视为队列名的多个消费实现**

需要配合 @RabbitHandler

```java
@RabbitListener(queues = "queue.order")
public class PayOrderUpdateListener {
	//监听队列的消费方法之一
	@RabbitHandler
	public void handlerMsg(String msg) {
	}
}

```

注解可选参数: @RabbitListener(isDefault = true)，这个参数值得注意,`isDefault = true`

```java
@RabbitHandler(queues = "queue.order", isDefault = true)
public void handleMsg(String msg) {
}

```

### 二、 消息的封装类Message ，及解析规则

```java
@RabbitHandler(queues = "queue.order")
public void handleMsg(Message msg) {
}

```

- 关键属性`content_type`, 用于约束body内容

1. [http://localhost:15672](http://localhost:15672/) 进入RabbitMQ的控制台
2. 设置消息的`content_type`

![202303132010063](@RabbitListener%E7%9A%84%E4%BD%BF%E7%94%A8%E5%8F%8A%E9%80%A0%E6%88%90%E6%8A%A5%E9%94%99%E6%AD%BB%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%88%86%E6%9E%90.assets/202303132010063.png)

3. `content_type` 的常见取值

|             content_type             |          handler的可接收形参          |
| :----------------------------------: | :-----------------------------------: |
|              text/plain              |                String                 |
|           application/json           |                  `/`                  |
| application/x-java-serialized-object |                实体类                 |
|       application/octet-stream       |                byte[]                 |
|                未指定                |                byte[]                 |
|                未指定                | org.springframework.amqp.core.Message |

未指定 `content_type` 的时候，可以用byte[]接收，
值得一提的是 `Message` 可用于接收所有可能的序列化场景，都不会报错。

### 异常捕获

先分析错误，再分析死循环

![202303132012889](@RabbitListener%E7%9A%84%E4%BD%BF%E7%94%A8%E5%8F%8A%E9%80%A0%E6%88%90%E6%8A%A5%E9%94%99%E6%AD%BB%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%88%86%E6%9E%90.assets/202303132012889.png)

**意思是消费者已经准备消费，队列里存在消息，但是找不到指定的消费实现方法来处理。**

#### 异常1问题：为什么会找不到消费实现？

1. `@RabbitListener` 或 `@RabbitHandler` 配置出错
   很大原因是取决于`content_type` 的配置和 方法的`形参`。
   如果通过客户端放入队列中**有个content_type为空的的消息，@RabbitListener只有形参为String 的Handler，是无法对应上消费实现的**。
2. `@RabbitHandler 没有使用可选参数isDefault`
   **消费者找不到任何一个消费实现，就回去找isDefault = true 的 handler，类似一个兜底策略。**

##### 异常1问题：处理思路

1. 使用`Message` 作为方法形参
2. 尽量将`@RabbitListener` 放在类上, 使用`@RabbitHandler(isDefault = true)` 做兜底策略

##### 异常1分析 ：死循环分析

这是一种应用级别的死循环，消息找不到**消费实现**，一直重试直到**找到消费实现**。这种死循环原因是配置失误，要在源头避免，测试阶段就要消灭。【找到消除这种死循环的方法再来填坑】。**另外一种**必须处理的死循环是已经找到消费实现，但是在消费的过程中造成死循环，见异常2：

#### 【异常2】：消费过程中抛出未捕获Exception

通常是**业务逻辑导致的异常**如`NullPointerException`，无脑的做法是`try-catch`，处理不当依旧会造成死循环。

##### 异常2问题：`try-catch`后仍然会死循环

[详细分析](https://www.jianshu.com/p/090ed51006d5)
这里简要概括下：`RabbitMQ` 默认的异常策略是`不断重试`，除非抛出了`fatal`类型的异常，这种异常类型如下

|             **异常类**              |
| :---------------------------------: |
|     MessageConversionException      |
|     MessageConversionException      |
|   MethodArgumentNotValidException   |
| MethodArgumentTypeMismatchException |
|        NoSuchMethodException        |
|         ClassCastException          |

类似于`NullPointerException`，是不再抛弃重试的范围的，也就是不主动捕获并处理，`RabbitMQ`会一直尝试消费该消息，导致死循环，从而使大量CPU资源被占用。

##### 异常2处理： 终止死循环

最简单的办法就是在`catch`语句块中抛出`fatal` 类型的异常，如

```java
    @RabbitHandler
    public void handlerMsg(String msg) {
        try { // try 住语句块 抛出致命异常
            //1.获取字符串转成map
            Map<String, String> map = JSON.parseObject(msg, Map.class);
            String out_trade_no = map.get("out_trade_no");
            if (map != null && map.get("return_code").equalsIgnoreCase("SUCCESS")) {
            }
        } catch (Exception e) {
            e.printStackTrace();
            // todo  解决消息队列重复试错的bug 抛出一个致命异常就会抛弃消费这个消息
            throw new MessageConversionException("消息消费失败，移出消息队列，不再试错");
        }
    }

```

**更健壮的方法：使用异常处理器 + 死信队列**

## 解决方案

1. 测试阶段暴力抛出`fatal`异常，终止死循环
2. 线上不暴力抛出`fatal`异常，先设置重试次数，[详细参数](https://www.jianshu.com/p/8614ff9ff0f1)
3. 超过重试次数，把消息加入死信队列，或者持久化到db，从消息队列暂时移除
4. 根据失败的消息做预警，`微信通知API` 或者其他方式触发人工干预