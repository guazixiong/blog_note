# Redis如何实现分布式锁

## 什么是分布式锁

分布式锁，顾名思义，就是分布式项目开发中用到的锁，可以用来控制分布式系统之间同步访问共享资源，一般来说，分布式锁需要满足的特性有这么几点：

- **互斥性**：在任何时刻，对于同一条数据，只有一台应用可以获取到分布式锁；
- **高可用性**：在分布式场景下，一小部分服务器宕机不影响正常使用，这种情况就需要将提供分布式锁的服务以集群的方式部署；
- **防止锁超时**：如果客户端没有主动释放锁，服务器会在一段时间之后自动释放锁，防止客户端宕机或者网络不可达时产生死锁；
- **独占性**：加锁解锁必须由同一台服务器进行，也就是锁的持有者才可以释放锁，不能出现你加的锁，别人给你解锁了；

业界里可以实现分布式锁效果的工具很多，但操作无非这么几个：**加锁、解锁、防止锁超时**。

## 实现锁的命令

先介绍下Redis的几个命令，

1、SETNX，用法是`SETNX key value`

SETNX是『 SET if Not eXists』(如果不存在，则 SET)的简写，设置成功就返回1，否则返回0。

![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232201981.png)

`setnx`用法

可以看出，当把**key**为**lock**的值设置为"Java"后，再设置成别的值就会失败，看上去很简单，也好像独占了锁，但有个致命的问题，就是**key**没有过期时间，这样一来，除非手动删除key或者获取锁后设置过期时间，不然其他线程永远拿不到锁。

既然这样，我们给key加个过期时间总可以吧，直接让线程获取锁的时候执行两步操作：

```bash
`SETNX Key 1`
`EXPIRE Key Seconds`
```

这个方案也有问题，因为获取锁和设置过期时间分成两步了，不是原子性操作，有可能**获取锁成功但设置时间失败**，那样不就白干了吗。

不过也不用急，这种事情Redis官方早为我们考虑到了，所以就引出了下面这个命令

2、SETEX，用法`SETEX key seconds value`

将值 `value` 关联到 `key` ，并将 `key` 的生存时间设为 `seconds` (以秒为单位)。如果 `key` 已经存在，SETEX 命令将覆写旧值。

这个命令类似于以下两个命令：

```autohotkey
`SET key value`
`EXPIRE key seconds  # 设置生存时间`
```

这两步动作是原子性的，会在同一时间完成。

![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232201038.png)

setex用法

3、PSETEX ，用法`PSETEX key milliseconds value`

这个命令和SETEX命令相似，但它以毫秒为单位设置 `key` 的生存时间，而不是像SETEX命令那样，以秒为单位。

不过，从Redis 2.6.12 版本开始，SET命令可以通过参数来实现和SETNX、SETEX、PSETEX 三个命令相同的效果。

就比如这条命令

```bash
`SET key value NX EX seconds` 
```

加上NX、EX参数后，效果就相当于SETEX，这也是Redis获取锁写法里面最常见的。

## 怎么释放锁

释放锁的命令就简单了，直接删除key就行，但我们前面说了，因为分布式锁必须由锁的持有者自己释放，所以我们必须先确保当前释放锁的线程是持有者，没问题了再删除，这样一来，就变成两个步骤了，似乎又违背了原子性了，怎么办呢？

不慌，我们可以用lua脚本把两步操作做拼装，就好像这样：

```lua
`if redis.call("get",KEYS[1]) == ARGV[1]`
`then`
 `return redis.call("del",KEYS[1])`
`else`
 `return 0`
`end`
```

> - KEYS[1]) ： 加锁的key
> - ARGV[1] ： key的生存时间，默认为30秒
> - ARGV[2] ： 加锁的客户端ID (UUID.randomUUID()） + “:” + threadId)

防止持有过期锁的线程，或者其他线程误删现有锁的情况出现。

## Redis分布式锁关键词

- **基于NIO的Netty框架，生产环境使用分布式锁**
- **redisson加锁：lua脚本加锁（其他客户端自旋）**
- **自动延时机制(看门狗)**：启动watch dog，**后台线程，每隔10秒检查一下客户端1还持有锁key，会不断的延长锁key的生存时间**
- **可重入锁机制**：第二个if判断 ，myLock :{“8743c9c0-0795-4907-87fd-6c719a6b4586:1”:2 }
- 释放锁：无锁直接返回；有锁不是我加的，返回；有锁是我加的，执行hincrby -1,当重入锁减完才执行del操作
- Redis使用同一个Lua解释器来执行所有命令，**Redis保证以一种原子性的方式来执行脚本：当lua脚本在执行的时候，不会有其他脚本和命令同时执行**，这种语义类似于 MULTI/EXEC。从别的客户端的视角来看，一个lua脚本要么不可见，要么已经执行完.

## 一、 Redisson使用

Redisson是架设在Redis基础上的一个Java驻内存数据网格（In-Memory Data Grid）。
Redisson在**基于NIO的Netty框架**上，生产环境使用分布式锁。

### pom依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>2.7.0</version>
</dependency>
```

### 配置Redission

```java
public class RedissonManager {
    private static Config config = new Config();
    //声明redisso对象
    private static Redisson redisson = null;

    //实例化redisson
    static {
        config.useClusterServers()
                // 集群状态扫描间隔时间，单位是毫秒
                .setScanInterval(2000)
                //cluster方式至少6个节点(3主3从，3主做sharding，3从用来保证主宕机后可以高可用)
                .addNodeAddress("redis://127.0.0.1:6379")
                .addNodeAddress("redis://127.0.0.1:6380")
                .addNodeAddress("redis://127.0.0.1:6381")
                .addNodeAddress("redis://127.0.0.1:6382")
                .addNodeAddress("redis://127.0.0.1:6383")
                .addNodeAddress("redis://127.0.0.1:6384");
        //得到redisson对象
        redisson = (Redisson) Redisson.create(config);
    }

    //获取redisson对象的方法
    public static Redisson getRedisson() {
        return redisson;
    }
}
```

### 锁的获取和释放

```java
public class DistributedRedisLock {
    //从配置类中获取redisson对象
    private static Redisson redisson = RedissonManager.getRedisson();
    private static final String LOCK_TITLE = "redisLock_";

    //加锁
    public static boolean acquire(String lockName) {
        //声明key对象
        String key = LOCK_TITLE + lockName;
        //获取锁对象
        RLock mylock = redisson.getLock(key);
        //加锁，并且设置锁过期时间3秒，防止死锁的产生  uuid+threadId
        mylock.lock(2, 3, TimeUtil.SECOND);
        //加锁成功
        return true;
    }

    //锁的释放
    public static void release(String lockName) {
        //必须是和加锁时的同一个key
        String key = LOCK_TITLE + lockName;
        //获取所对象
        RLock mylock = redisson.getLock(key);
        //释放锁（解锁）
        mylock.unlock();
    }
}
```

### 业务逻辑中使用分布式锁

```java
public String discount() throws IOException{
    String key = "lock001";
    //加锁
    DistributedRedisLock.acquire(key);
    //执行具体业务逻辑
    // dosoming
    //释放锁
    DistributedRedisLock.release(key);
    //返回结果
    return soming;
 }
```

## 二、Redisson分布式锁的实现原理

![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232149990.png)

### 3.1 加锁机制

1. 如果该客户端面对的是一个redis cluster集群，他首先会根据hash节点选择一台机器。
2. 发送lua脚本到redis服务器上，脚本如下:

```lua
//exists',KEYS[1])==0 不存在，没锁
"if (redis.call('exists',KEYS[1])==0) then "+       --看有没有锁
  // 命令：hset,1：第一回
	"redis.call('hset',KEYS[1],ARGV[2],1) ; "+       --无锁 加锁 
	// 配置锁的生命周期 
	"redis.call('pexpire',KEYS[1],ARGV[1]) ; "+      
	"return nil; end ;" +

//可重入操作，判断是不是我加的锁
"if (redis.call('hexists',KEYS[1],ARGV[2]) ==1 ) then "+  --我加的锁
   //hincrby 在原来的锁上加1
	"redis.call('hincrby',KEYS[1],ARGV[2],1) ; "+  --重入锁
	"redis.call('pexpire',KEYS[1],ARGV[1]) ; "+  
	"return nil; end ;" +

//否则，锁存在，返回锁的有效期，决定下次执行脚本时间
"return redis.call('pttl',KEYS[1]) ;"  --不能加锁，返回锁的时间
```

**lua的作用：保证这段复杂业务逻辑执行的原子性。**

> lua的解释：
>
> - KEYS[1]) ： 加锁的key
>
> - ARGV[1] ： key的生存时间，默认为30秒
> - ARGV[2] ： 加锁的客户端ID (UUID.randomUUID()） + “:” + threadId)
>
> 第一段if判断语句，就是用“exists myLock”命令判断一下，如果你要加锁的那个锁key不存在的话，你就进行加锁。
> 如何加锁呢？很简单，用下面的命令：
>
> - `hset myLock 8743c9c0-0795-4907-87fd-6c719a6b4586:1 1`
>
> 通过这个命令设置一个hash数据结构，这行命令执行后，会出现一个类似下面的数据结构：
>
> - `myLock :{“8743c9c0-0795-4907-87fd-6c719a6b4586:1”:1 }`
>
> 上述就代表“8743c9c0-0795-4907-87fd-6c719a6b4586:1”这个客户端对“myLock”这个锁key完成了加锁。
>
> 接着会执行“pexpire myLock 30000”命令，设置myLock这个锁key的生存时间是30秒。

#### 锁互斥机制

那么在这个时候，如果**客户端2**来尝试加锁，执行了同样的一段lua脚本，会咋样呢？

很简单，**第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在了**。

接着**第二个if判断**，判断一下，myLock锁key的hash数据结构中，是否包含客户端2的ID，但是明显不是的，

因为那里包含的是客户端1的ID。

所以，**客户端2会获取到pttl myLock返回的一个数字，这个数字代表了myLock这个锁key的剩余生存时间**。

比如还剩15000毫秒的生存时间。

**此时客户端2会进入一个while循环，不停的尝试加锁**。

#### 自动延时机制(看门狗)

只要客户端1一旦加锁成功，就会**启动一个watch dog看门狗**，他是一个**后台线程**，会**每隔10秒检查一下**，如果客户端1还持有锁key，那么就会**不断的延长锁key的生存时间**。

#### 可重入锁机制

第一个if判断 肯定不成立，“exists myLock”会显示锁key已经存在了。

第二个if判断 会成立，因为myLock的hash数据结构中包含的那个ID，就是客户端1的那个ID，也就是`“8743c9c0-0795-4907-87fd-6c719a6b4586:1”`

此时就会执行可重入加锁的逻辑，他会用：`incrby myLock 8743c9c0-0795-4907-87fd-6c71a6b4586:1 1`

通过这个命令，对客户端1的加锁次数，累加1。数据结构会变成：`myLock :{“8743c9c0-0795-4907-87fd-`

`6c719a6b4586:1”:2 }`

### 2.2 释放锁机制

执行lua脚本如下：

```lua
# 如果key已经不存在，说明已经被解锁，直接发布（publish）redis消息（无锁，直接返回）
"if (redis.call('exists', KEYS[1]) == 0) then " +
            "redis.call('publish', KEYS[2], ARGV[1]); " +
            "return 1; " +
          "end;" +
# key和field不匹配，说明当前客户端线程没有持有锁，不能主动解锁。 不是我加的锁 不能解锁 （有锁不是我加的，返回）
          "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
            "return nil;" +
          "end; " +
# 将value减1 （有锁是我加的，进行hincrby -1 ）
          "local counter = redis.call('hincrby', KEYS[1], ARGV[3],-1); " +
# 如果counter>0说明锁在重入，不能删除key
          "if (counter > 0) then " +
            "redis.call('pexpire', KEYS[1], ARGV[2]); " +
            "return 0; " +
# 删除key并且publish 解锁消息
					# 可重入锁减完了，进行del操作
          "else " + 
            "redis.call('del', KEYS[1]); " + #删除锁
            "redis.call('publish', KEYS[2], ARGV[1]); " +
            "return 1; "+
             "end; " +
             "return nil;",

```

- – KEYS[1] ：需要加锁的key，这里需要是字符串类型。
- – KEYS[2] ：redis消息的ChannelName，一个分布式锁对应唯一的一个channelName: “redisson_lockchannel{” + getName() + “}”
- – ARGV[1] ：reids消息体，这里只需要一个字节的标记就可以，主要标记redis的key已经解锁，再结合 redis的Subscribe，能唤醒其他订阅解锁消息的客户端线程申请锁。
- – ARGV[2] ：锁的超时时间，防止死锁
- – ARGV[3] ：锁的唯一标识，也就是刚才介绍的 id（UUID.randomUUID()） + “:” + threadId

如果执行lock.unlock()，就可以释放分布式锁，此时的业务逻辑也是非常简单的。其实说白了，就是每次都对myLock数据结构中的那个加锁次数减1。如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：“del myLock”命令，从redis里删除这个key。然后呢，另外的客户端2就可以尝试完成加锁了。

#### 分布式锁特性

- **互斥性**:任意时刻，只能有一个客户端获取锁，不能同时有两个客户端获取到锁。
- **同一性**:锁只能被持有该锁的客户端删除，不能由其它客户端删除。
- **可重入性**:持有某个锁的客户端可继续对该锁加锁，实现锁的续租
- **容错性**:锁失效后（超过生命周期）自动释放锁（key失效），其他客户端可以继续获得该锁，防止死锁

#### 分布式锁的实际应用

- 数据并发竞争:利用分布式锁可以将处理串行化，前面已经讲过了。

- 防止库存超卖

  ![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232155426.png)



- 订单1下单前会先查看库存，库存为10，所以下单5本可以成功；

- 订单2下单前会先查看库存，库存为10，所以下单8本可以成功；

  订单1和订单2 同时操作，共下单13本，但库存只有10本，显然库存不够了，这种情况称为库存超卖。
  可以采用分布式锁解决这个问题。
  ![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232156709.png)订单1和订单2都从Redis中获得分布式锁(setnx)，谁能获得锁谁进行下单操作，这样就把订单系统下单的顺序串行化了，就不会出现超卖的情况了。伪码如下：

```java
//加锁并设置有效期
if(redis.lock("RDL",200)){
  //判断库存
  if (orderNum<getCount()){
  //加锁成功 ,可以下单
  order(5);
  //释放锁
  redis,unlock("RDL");
 }  
}
```

注意此种方法会降低处理效率，这样不适合秒杀的场景，秒杀可以使用CAS和Redis队列的方式

## 分布式锁的缺陷

### **客户端长时间阻塞导致锁失效问题**

**客户端1得到了锁，因为网络问题或者GC等原因导致长时间阻塞，然后业务程序还没执行完锁就过期了，这时候客户端2也能正常拿到锁，可能会导致线程安全的问题。**

![preview](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232205533.png)

客户端长时间阻塞

那么该如何防止这样的异常呢？我们先不说解决方案，介绍完其他的缺陷后再来讨论。

### **redis服务器时钟漂移问题**

如果redis服务器的机器时钟发生了向前跳跃，就会导致这个key过早超时失效，比如:

客户端1拿到锁后，key的过期时间是12:02分，但redis服务器本身的时钟比客户端快了2分钟，导致key在12:00的时候就失效了，这时候，如果客户端1还没有释放锁的话，就可能导致多个客户端同时持有同一把锁的问题。

### 单点实例安全问题

如果redis是单master模式的，当这台机宕机的时候，那么所有的客户端都获取不到锁了，为了提高可用性，

可能就会给这个master加一个slave，但是因为**redis的主从同步**是异步进行的，可能会出现**客户端1设置完锁**

**后，master挂掉，slave提升为master，因为异步复制的特性，客户端1设置的锁丢失了，这时候客户端2设置锁也能够成功，导致客户端1和客户端2同时拥有锁。**

为了解决Redis单点问题，redis的作者提出了**RedLock**算法。

## RedLock算法

该算法的实现前提在于**Redis必须是多节点部署**的，可以有效防止单点故障，具体的实现思路是这样的：

1. 获取当前时间戳（ms）；
2. 先设定key的有效时长（TTL），超出这个时间就会自动释放，然后client（客户端）尝试使用相同的key和value对所有redis实例进行设置，每次链接redis实例时设置一个比TTL短很多的超时时间，这是为了不要过长时间等待已经关闭的redis服务。并且试着获取下一个redis实例。

​		比如：TTL（也就是过期时间）为5s，那获取锁的超时时间就可以设置成50ms，所以如果50ms内无法获		取锁，就放弃获取这个锁，从而尝试获取下个锁；

3. client通过获取所有能获取的锁后的时间减去第一步的时间，还有redis服务器的时钟漂移误差，然后这个时间差要小于TTL时间并且成功设置锁的实例数>= N/2 + 1（N为Redis实例的数量），那么加锁成功

​		比如TTL是5s，连接redis获取所有锁用了2s，然后再减去时钟漂移（假设误差是1s左右），那么锁的真		正有效时长就只有2s了；

4. **如果客户端由于某些原因获取锁失败，便会开始解锁所有redis实例**。

根据这样的算法，我们假设有5个Redis实例的话，那么client只要获取其中3台以上的锁就算是成功了，用流程图演示大概就像这样： 

![img](redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.assets/202303232207330.png)

key有效时长

好了，算法也介绍完了，从设计上看，毫无疑问，RedLock算法的思想主要是为了有效防止Redis单点故障的问题，而且在设计TTL的时候也考虑到了服务器时钟漂移的误差，让分布式锁的安全性提高了不少。

但事实真的是这样吗？反正我个人的话感觉效果一般般，

首先第一点，我们可以看到，在RedLock算法中，锁的有效时间会减去连接Redis实例的时长，如果这个过程因为网络问题导致耗时太长的话，那么最终留给锁的有效时长就会大大减少，客户端访问共享资源的时间很短，很可能程序处理的过程中锁就到期了。而且，锁的有效时间还需要减去服务器的时钟漂移，但是应该减多少合适呢，要是这个值设置不好，很容易出现问题。

然后第二点，这样的算法虽然考虑到用多节点来防止Redis单点故障的问题，但但如果有节点发生崩溃重启的话，还是有可能出现多个客户端同时获取锁的情况。

假设一共有5个Redis节点：A、B、C、D、E，客户端1和2分别加锁

1. 客户端1成功锁住了A，B，C，获取锁成功（但D和E没有锁住）。
2. 节点C的master挂了，然后锁还没同步到slave，slave升级为master后丢失了客户端1加的锁。
3. 客户端2这个时候获取锁，锁住了C，D，E，获取锁成功。

这样，客户端1和客户端2就同时拿到了锁，程序安全的隐患依然存在。除此之外，如果这些节点里面某个节点发生了时间漂移的话，也有可能导致锁的安全问题。

所以说，虽然通过多实例的部署提高了可用性和可靠性，**但RedLock并没有完全解决Redis单点故障存在的隐患，也没有解决时钟漂移以及客户端长时间阻塞而导致的锁超时失效存在的问题，锁的安全性隐患依然存在。**

## 结论

有人可能要进一步问了，那该怎么做才能保证锁的绝对安全呢？

对此我只能说，鱼和熊掌不可兼得，我们之所以用Redis作为分布式锁的工具，很大程度上是因为Redis本身效率高且单进程的特点，即使在高并发的情况下也能很好的保证性能，但很多时候，性能和安全不能完全兼顾，如果你一定要保证锁的安全性的话，可以用其他的中间件如db、zookeeper来做控制，这些工具能很好的保证锁的安全，但性能方面只能说是差强人意，否则大家早就用上了。

一般来说，用Redis控制共享资源并且还要求数据安全要求较高的话，最终的保底方案是对业务数据做幂等控制，这样一来，即使出现多个客户端获得锁的情况也不会影响数据的一致性。当然，也不是所有的场景都适合这么做，具体怎么取舍就需要各位看官自己处理啦，毕竟，没有完美的技术，只有适合的才是最好的。

> Redisson分布式锁的使用（推荐使用）_redisson分布式锁使用_穿城大饼的博客-CSDN博客
> https://blog.csdn.net/chuanchengdabing/article/details/121210426
>
> 面试官：你真的了解Redis分布式锁吗？ - 技术进阶之路 - SegmentFault 思否
> https://segmentfault.com/a/1190000038988087

