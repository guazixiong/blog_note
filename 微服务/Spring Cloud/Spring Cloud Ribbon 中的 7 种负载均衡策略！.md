# Spring Cloud Ribbon 中的 7 种负载均衡策略 + 自定义算法

负载均衡通器常有两种实现手段，一种是服务端负载均衡器，另一种是客户端负载均衡器，而我们今天的主角 Ribbon 就属于后者——客户端负载均衡器。

**服务端负载均衡器的问题是，它提供了更强的流量控制权，但无法满足不同的消费者希望使用不同负载均衡策略的需求，而使用不同负载均衡策略的场景确实是存在的，所以客户端负载均衡就提供了这种灵活性。** 然而客户端负载均衡也有其缺点，如果配置不当，可能会导致服务提供者出现热点，或者压根就拿不到任何服务的情况，所以我们本文就来了解一下这 7 种内置负载均衡策略的具体规则。

- **轮询策略**
- **权重策略**
- **随机策略**
- **最小连接数策略**
- **重试策略**
- **可用性敏感策略**
- **区域性敏感策略**
- **自定义轮询策略**

## Ribbon 介绍

Ribbon 是 Spring Cloud 技术栈中非常重要的基础框架，它为 Spring Cloud 提供了负载均衡的能力，比如 Fegin 和 OpenFegin 都是基于 Ribbon 实现的，就连 Nacos 中的负载均衡也使用了 Ribbon 框架。

Ribbon 框架的强大之处在于，它不仅内置了 7 种负载均衡策略，同时还支持用户自定义负载均衡策略，所以其开放性和便利性也是它得以流行的主要原因。

服务端负载均衡器和客户端负载均衡器的区别如下图所示：

 ![image.png](Spring%20Cloud%20Ribbon%20%E4%B8%AD%E7%9A%84%207%20%E7%A7%8D%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5%EF%BC%81.assets/202303181800777.webp) 客户端负载均衡器的实现原理是通过注册中心，如 Nacos，将可用的服务列表拉取到本地（客户端），再通过客户端负载均衡器（设置的负载均衡策略)获取到某个服务器的具体 ip 和端口，然后再通过 Http 框架请求服务并得到结果，其执行流程如下图所示：

![image.png](Spring%20Cloud%20Ribbon%20%E4%B8%AD%E7%9A%84%207%20%E7%A7%8D%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5%EF%BC%81.assets/202303181800375.webp)

## 负载均衡设置

rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标 ，每次服务重启动后rest接口计数从1开始.

以 Nacos 中的 Ribbon 负载均衡设置为例，在配置文件 application.yml 中设置如下配置即可：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #设置负载均衡策略
```

因为 Nacos 中已经内置了 Ribbon，所以在实际项目开发中无需再添加 Ribbon 依赖了，这一点我们在 Nacos 的依赖树中就可以看到，如下图所示： ![image.png](Spring%20Cloud%20Ribbon%20%E4%B8%AD%E7%9A%84%207%20%E7%A7%8D%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5%EF%BC%81.assets/202303181801516.webp) Ribbon 默认的负载均衡策略是轮询模式，我们配置 3 个服务提供者的执行结果如下图所示：

 ![轮询.gif](Spring%20Cloud%20Ribbon%20%E4%B8%AD%E7%9A%84%207%20%E7%A7%8D%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5%EF%BC%81.assets/202303181801631.webp) 然后，我们再将 Ribbon 负载均衡策略设置为随机模式，配置内容如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #设置随机负载均衡
复制代码
```

重启客户端，执行结果如下图所示：

 ![随机策略.gif](Spring%20Cloud%20Ribbon%20%E4%B8%AD%E7%9A%84%207%20%E7%A7%8D%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5%EF%BC%81.assets/202303181801578.webp)

## 7种负载均衡策略

### 1.轮询策略

```bash
# 每次服务重启动后rest接口计数从1开始.
rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标


# 如8001 + 8002 组合为集群,共计2台机器,集群总数为2,
当总请求数为1时,1%2 = 1,对应下标位置未1,获取服务地址${ip}:8001
当总请求数为2时,2%2 = 0,对应下标位置未1,获取服务地址${ip}:8002
当总请求数为3时,3%2 = 1,对应下标位置未1,获取服务地址${ip}:8001
当总请求数为4时,4%2 = 0,对应下标位置未1,获取服务地址${ip}:8002
当总请求数为5时,5%2 = 1,对应下标位置未1,获取服务地址${ip}:8001
当总请求数为6时,6%2 = 0,对应下标位置未1,获取服务地址${ip}:8002
......
```

轮询策略：RoundRobinRule，按照一定的顺序依次调用服务实例。比如一共有 3 个服务，第一次调用服务 1，第二次调用服务 2，第三次调用服务3，依次类推。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #设置负载均衡
```

自定义轮询算法

```java
public class MyLoadBalancer implements com.atguigu.springcloud.lb.MyLoadBalancer {
    private final AtomicInteger atomicInteger = new AtomicInteger(0);

    /**
     * @author pangdi
     * @date 2023/03/18 10:07
     * @description 判断时第几次访问
     */
    public final int getAndIncrement() {
        int current;
        int next = 0;
        do {
            current = atomicInteger.get();
            // 防止越界
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        } while (!atomicInteger.compareAndSet(current, next));

        System.out.println("*****第几次访问，次数next: " + next);
        return next;
    }

    /**
     * 负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标， 每次服务重启动后rest接口计数从1开始。1
     *
     * @param serviceInstances 服务实例
     * @return ServiceInstance 服务实例
     * @author pangdi
     * @date 2023/03/18 9:51
     */
    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index = getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }

}
```

### 2.权重策略

权重策略：WeightedResponseTimeRule，根据每个服务提供者的响应时间分配一个权重，响应时间越长，权重越小，被选中的可能性也就越低。 它的实现原理是，刚开始使用轮询策略并开启一个计时器，每一段时间收集一次所有服务提供者的平均响应时间，然后再给每个服务提供者附上一个权重，权重越高被选中的概率也越大。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

### 3.随机策略

随机策略：RandomRule，从服务提供者的列表中随机选择一个服务实例。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #设置负载均衡
```

### 4.最小连接数策略

最小连接数策略：BestAvailableRule，也叫最小并发数策略，它是遍历服务提供者列表，选取连接数最小的⼀个服务实例。如果有相同的最小连接数，那么会调用轮询策略进行选取。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.BestAvailableRule #设置负载均衡
```

### 5.重试策略

重试策略：RetryRule，按照轮询策略来获取服务，如果获取的服务实例为 null 或已经失效，则在指定的时间之内不断地进行重试来获取服务，如果超过指定时间依然没获取到服务实例则返回 null。 此策略的配置设置如下：

```yaml
ribbon:
  ConnectTimeout: 2000 # 请求连接的超时时间
  ReadTimeout: 5000 # 请求处理的超时时间
springcloud-nacos-provider: # nacos 中的服务 id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #设置负载均衡
```

### 6.可用性敏感策略

可用敏感性策略：AvailabilityFilteringRule，先过滤掉非健康的服务实例，然后再选择连接数较小的服务实例。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.AvailabilityFilteringRule
```

### 7.区域敏感策略

区域敏感策略：ZoneAvoidanceRule，根据服务所在区域（zone）的性能和服务的可用性来选择服务实例，在没有区域的环境下，该策略和轮询策略类似。 此策略的配置设置如下：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.ZoneAvoidanceRule
```

## 总结

Ribbon 为客户端负载均衡器，相比于服务端负载均衡器的统一负载均衡策略来说，它提供了更多的灵活

性。Ribbon 内置了 7 种负载均衡策略：轮询策略、权重策略、随机策略、最小连接数策略、重试策略、可

用性敏感策略、区域性敏感策略，并且用户可以通过继承 RoundRibbonRule 来实现自定义负载均衡策略。