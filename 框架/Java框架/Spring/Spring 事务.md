# Spring 事务

- 1. 什么是事务

- 2. Spring 中的事务

- 2.1 两种用法

- 2.2 三大基础设施

- 3. 编程式事务

- 4. 声明式事务

- 4.1 XML 配置

- 4.2 Java 配置

- 4.3 混合配置

- 5. 事务属性

- 5.1 隔离性

- 5.2 传播性

- 5.3 回滚规则

- 5.4 是否只读

- 5.5 超时时间

- 6. 注意事项

- 7. 小结

事务的重要性不言而喻，Spring 对事务也提供了非常丰富的支持，各种支持的属性应有尽有。

然而很多小伙伴知道，这里有两个属性特别绕：

- 隔离性(事务的隔离级别,脏读,幻读.不可重复读)
- 传播性(事务的一致性,是否嵌套事务)

## 1. 什么是事务

**数据库事务是指作为单个逻辑工作单元执行的一系列操作，这些操作要么一起成功，要么一起失败，是一个不可分割的工作单元。**

在我们日常工作中，涉及到事务的场景非常多，一个 service 中往往需要调用不同的 dao 层方法，这些方法要么同时成功要么同时失败，我们需要在 service 层确保这一点。

说到事务最典型的案例就是转账了：

张三要给李四转账 500 块钱，这里涉及到两个操作，从张三的账户上减去 500 块钱，给李四的账户添加 500 块钱，这两个操作要么同时成功要么同时失败，如何确保他们同时成功或者同时失败呢？答案就是事务。

事务有四大特性（ACID）：

![](Spring%20%E4%BA%8B%E5%8A%A1.assets/1695110537885-d7afe874-4cbe-4a92-8001-8673dae54f5e.png)

- **原子性（Atomicity）：** 一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- **一致性（Consistency）：** 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器、级联回滚等。
- **隔离性（Isolation）：** 数据库允许多个并发事务同时对其数据进行读写和修改，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read Uncommitted）、提交读（Read Committed）、可重复读（Repeatable Read）和串行化（Serializable）。
- **持久性（Durability）:** 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

这就是事务的四大特性。

## 2. Spring 中的事务

### 2.1 两种用法

Spring 作为 Java 开发中的基础设施，对于事务也提供了很好的支持，总体上来说，Spring 支持两种类型的事务，声明式事务和编程式事务。

编程式事务类似于 Jdbc 事务的写法，需要将事务的代码嵌入到业务逻辑中，这样代码的耦合度较高，而声明式事务通过 AOP 的思想能够有效的将事务和业务逻辑代码解耦，因此在实际开发中，声明式事务得到了广泛的应用，而编程式事务则较少使用，考虑到文章内容的完整，本文对两种事务方式都会介绍。

### 2.2 三大基础设施

Spring 中对事务的支持提供了三大基础设施，我们先来了解下。

1. **PlatformTransactionManager**
2. **TransactionDefinition**
3. **TransactionStatus**

这三个核心类是 Spring 处理事务的核心类。

#### 2.2.1 PlatformTransactionManager

PlatformTransactionManager 是事务处理的核心，它有诸多的实现类，如下：

![](Spring%20%E4%BA%8B%E5%8A%A1.assets/1695110537948-1b9eac2c-5ac1-49a5-a465-d1be8d4142dd.png)

PlatformTransactionManager 的定义如下：

```
public interface PlatformTransactionManager {
 TransactionStatus getTransaction(@Nullable TransactionDefinition definition);
 void commit(TransactionStatus status) throws TransactionException;
 void rollback(TransactionStatus status) throws TransactionException;
}
```

可以看到 PlatformTransactionManager 中定义了基本的事务操作方法，这些事务操作方法都是平台无关的，具体的实现都是由不同的子类来实现的。

这就像 JDBC 一样，SUN 公司制定标准，其他数据库厂商提供具体的实现。这么做的好处就是我们 Java 程序员只需要掌握好这套标准即可，不用去管接口的具体实现。以 PlatformTransactionManager 为例，它有众多实现，如果你使用的是 JDBC 那么可以将 DataSourceTransactionManager 作为事务管理器；如果你使用的是 Hibernate，那么可以将 HibernateTransactionManager 作为事务管理器；如果你使用的是 JPA，那么可以将 JpaTransactionManager 作为事务管理器。DataSourceTransactionManager、HibernateTransactionManager 以及 JpaTransactionManager 都是 PlatformTransactionManager 的具体实现，但是我们并不需要掌握这些具体实现类的用法，我们只需要掌握好 PlatformTransactionManager 的用法即可。

PlatformTransactionManager 中主要有如下三个方法：

**1.getTransaction()**

getTransaction() 是根据传入的 TransactionDefinition 获取一个事务对象，TransactionDefinition 中定义了一些事务的基本规则，例如传播性、隔离级别等。

**2.commit()**

commit() 方法用来提交事务。

**3.rollback()**

rollback() 方法用来回滚事务。

#### 2.2.2 TransactionDefinition

TransactionDefinition 用来描述事务的具体规则，也称作事务的属性。事务有哪些属性呢？看下图：

![](Spring%20%E4%BA%8B%E5%8A%A1.assets/1695110537991-96d06bae-e2d6-4533-ab4f-f5992595063e.png)

可以看到，主要是五种属性：

1. 隔离性
2. 传播性
3. 回滚规则
4. 超时时间
5. 是否只读

这五种属性接下来松哥会和大家详细介绍。

TransactionDefinition 类中的方法如下：

![](Spring%20%E4%BA%8B%E5%8A%A1.assets/1695110537928-e7cab85d-1ffa-4703-8820-54672dfe6abd.png)

可以看到一共有五个方法：

1. getIsolationLevel()，获取事务的隔离级别
2. getName()，获取事务的名称
3. getPropagationBehavior()，获取事务的传播性
4. getTimeout()，获取事务的超时时间
5. isReadOnly()，获取事务是否是只读事务

TransactionDefinition 也有诸多的实现类，如下：

![](Spring%20%E4%BA%8B%E5%8A%A1.assets/1695110537823-f1306e30-ae19-4418-b243-9ce892eb87a5.png)

如果开发者使用了编程式事务的话，直接使用 DefaultTransactionDefinition 即可。

#### 2.2.3 TransactionStatus

TransactionStatus 可以直接理解为事务本身，该接口源码如下：

```
public interface TransactionStatus extends SavepointManager, Flushable {
 boolean isNewTransaction();
 boolean hasSavepoint();
 void setRollbackOnly();
 boolean isRollbackOnly();
 void flush();
 boolean isCompleted();
}
```

1. isNewTransaction() 方法获取当前事务是否是一个新事务。
2. hasSavepoint() 方法判断是否存在 savePoint()。
3. setRollbackOnly() 方法设置事务必须回滚。
4. isRollbackOnly() 方法获取事务只能回滚。
5. flush() 方法将底层会话中的修改刷新到数据库，一般用于 Hibernate/JPA 的会话，对如 JDBC 类型的事务无任何影响。
6. isCompleted() 方法用来获取是一个事务是否结束。

**这就是 Spring 中支持事务的三大基础设施。**

## 3. 编程式事务

我们先来看看编程式事务怎么玩。

通过 PlatformTransactionManager 或者 TransactionTemplate 可以实现编程式事务。如果是在 Spring Boot 项目中，这两个对象 Spring Boot 会自动提供，我们直接使用即可。但是如果是在传统的 SSM 项目中，则需要我们通过配置来提供这两个对象，松哥给一个简单的配置参考，如下（简单起见，数据库操作我们使用 JdbcTemplate）：

```
<bean class="org.springframework.jdbc.datasource.DriverManagerDataSource" id="dataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///spring_tran?serverTimezone=Asia/Shanghai"/>
    <property name="username" value="root"/>
    <property name="password" value="123"/>
</bean>
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<bean class="org.springframework.transaction.support.TransactionTemplate" id="transactionTemplate">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
<bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

有了这两个对象，接下来的代码就简单了：

```
@Service
public class TransferService {
    @Autowired
    JdbcTemplate jdbcTemplate;
    @Autowired
    PlatformTransactionManager txManager;

    public void transfer() {
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
        TransactionStatus status = txManager.getTransaction(definition);
        try {
            jdbcTemplate.update("update user set account=account+100 where username='zhangsan'");
            int i = 1 / 0;
            jdbcTemplate.update("update user set account=account-100 where username='lisi'");
            txManager.commit(status);
        } catch (DataAccessException e) {
            e.printStackTrace();
            txManager.rollback(status);
        }
    }
}
```

这段代码很简单，没啥好解释的，在 try...catch... 中进行业务操作，没问题就 commit，有问题就 rollback。如果我们需要配置事务的隔离性、传播性等，可以在 DefaultTransactionDefinition 对象中进行配置。

上面的代码是通过 PlatformTransactionManager 实现的编程式事务，我们也可以通过 TransactionTemplate 来实现编程式事务，如下：

```
@Service
public class TransferService {
    @Autowired
    JdbcTemplate jdbcTemplate;
    @Autowired
    TransactionTemplate tranTemplate;
    public void transfer() {
        tranTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                try {
                    jdbcTemplate.update("update user set account=account+100 where username='zhangsan'");
                    int i = 1 / 0;
                    jdbcTemplate.update("update user set account=account-100 where username='lisi'");
                } catch (DataAccessException e) {
                    status.setRollbackOnly();
                    e.printStackTrace();
                }
            }
        });
    }
}
```

直接注入 TransactionTemplate，然后在 execute 方法中添加回调写核心的业务即可，当抛出异常时，将当前事务标注为只能回滚即可。注意，execute 方法中，如果不需要获取事务执行的结果，则直接使用 TransactionCallbackWithoutResult 类即可，如果要获取事务执行结果，则使用 TransactionCallback 即可。

这就是两种编程式事务的玩法。

编程式事务由于代码入侵太严重了，因为在实际开发中使用的很少，我们在项目中更多的是使用声明式事务。

## 4. 声明式事务

声明式事务如果使用 XML 配置，可以做到无侵入；如果使用 Java 配置，也只有一个 @Transactional 注解侵入而已，相对来说非常容易。

**以下配置针对传统 SSM 项目（因为在 Spring Boot 项目中，事务相关的组件已经配置好了）：**

### 4.1 XML 配置

XML 配置声明式事务大致上可以分为三个步骤，如下：

1. 配置事务管理器

```
<bean class="org.springframework.jdbc.datasource.DriverManagerDataSource" id="dataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///spring_tran?serverTimezone=Asia/Shanghai"/>
    <property name="username" value="root"/>
    <property name="password" value="123"/>
</bean>
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

1. 配置事务通知

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="m3"/>
        <tx:method name="m4"/>
    </tx:attributes>
</tx:advice>
```

1. 配置 AOP

```
<aop:config>
    <aop:pointcut id="pc1" expression="execution(* org.javaboy.demo.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pc1"/>
</aop:config>
```

第二步和第三步中定义出来的方法交集，就是我们要添加事务的方法。

配置完成后，如下一些方法就自动具备事务了：

```
public class UserService {
    public void m3(){
        jdbcTemplate.update("update user set money=997 where username=?", "zhangsan");
    }
}
```

### 4.2 Java 配置

我们也可以使用 Java 配置来实现声明式事务：

```
@Configuration
@ComponentScan
//开启事务注解支持
@EnableTransactionManagement
public class JavaConfig {
    @Bean
    DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setPassword("123");
        ds.setUsername("root");
        ds.setUrl("jdbc:mysql:///test01?serverTimezone=Asia/Shanghai");
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        return ds;
    }

    @Bean
    JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

这里要配置的东西其实和 XML 中配置的都差不多，最最关键的就两个：

- 事务管理器 PlatformTransactionManager。
- @EnableTransactionManagement 注解开启事务支持。

配置完成后，接下来，哪个方法需要事务就在哪个方法上添加 @Transactional 注解即可，向下面这样：

```
@Transactional(noRollbackFor = ArithmeticException.class)
public void update4() {
    jdbcTemplate.update("update account set money = ? where username=?;", 998, "lisi");
    int i = 1 / 0;
}
```

当然这个稍微有点代码入侵，不过问题不大，日常开发中这种方式使用较多。当@Transactional 注解加在类上面的时候，表示该类的所有方法都有事务，该注解加在方法上面的时候，表示该方法有事务。

### 4.3 混合配置

也可以 Java 代码和 XML 混合配置来实现声明式事务，就是一部分配置用 XML 来实现，一部分配置用 Java 代码来实现：

假设 XML 配置如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd   http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--
    开启事务的注解配置，添加了这个配置，就可以直接在代码中通过 @Transactional 注解来开启事务了
    -->
    <tx:annotation-driven />

</beans>
```

那么 Java 代码中的配置如下：

```
@Configuration
@ComponentScan
@ImportResource(locations = "classpath:applicationContext3.xml")
public class JavaConfig {
    @Bean
    DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setPassword("123");
        ds.setUsername("root");
        ds.setUrl("jdbc:mysql:///test01?serverTimezone=Asia/Shanghai");
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        return ds;
    }

    @Bean
    JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

Java 配置中通过 @ImportResource 注解导入了 XML 配置，XML 配置中的内容就是开启 @Transactional 注解的支持，所以 Java 配置中省略了 @EnableTransactionManagement 注解。

这就是声明式事务的几种配置方式。好玩吧！

## 5. 事务属性

在前面的配置中，我们只是简单说了事务的用法，并没有和大家详细聊一聊事务的一些属性细节，那么接下来我们就来仔细捋一捋事务中的五大属性。

### 5.1 隔离性

首先就是事务的隔离性，也就是事务的隔离级别。

MySQL 中有四种不同的隔离级别，这四种不同的隔离级别在 Spring 中都得到了很好的支持。**Spring 中默认的事务隔离级别是 default，即数据库本身的隔离级别是啥就是啥，default 就能满足我们日常开发中的大部分场景**。

不过如果项目有需要，我们也可以调整事务的隔离级别。

调整方式如下：

#### 5.1.1 编程式事务隔离级别

如果是编程式事务，通过如下方式修改事务的隔离级别：

**TransactionTemplate**

```
transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_SERIALIZABLE);
```

TransactionDefinition 中定义了各种隔离级别。

**PlatformTransactionManager**

```
public void update2() {
    //创建事务的默认配置
    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    definition.setIsolationLevel(TransactionDefinition.ISOLATION_SERIALIZABLE);
    TransactionStatus status = platformTransactionManager.getTransaction(definition);
    try {
        jdbcTemplate.update("update account set money = ? where username=?;", 999, "zhangsan");
        int i = 1 / 0;
        //提交事务
        platformTransactionManager.commit(status);
    } catch (DataAccessException e) {
          e.printStackTrace();
        //回滚
        platformTransactionManager.rollback(status);
    }
}
```

这里是在 DefaultTransactionDefinition 对象中设置事务的隔离级别。

#### 5.1.2 声明式事务隔离级别

如果是声明式事务通过如下方式修改隔离级别：

XML：

```
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以 add 开始的方法，添加事务-->
        <tx:method name="add*"/>
        <tx:method name="insert*" isolation="SERIALIZABLE"/>
    </tx:attributes>
</tx:advice>
```

Java：

```
@Transactional(isolation = Isolation.SERIALIZABLE)
public void update4() {
    jdbcTemplate.update("update account set money = ? where username=?;", 998, "lisi");
    int i = 1 / 0;
}
```

关于事务的隔离级别，如果大家还不熟悉，可以参考松哥之前的文章：[四个案例看懂 MySQL 事务隔离级别](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247494689&idx=1&sn=59e5a30d342f9f1330830b9221aa45a8&scene=21#wechat_redirect)。

### 5.2 传播性

先来说说何谓事务的传播性：

事务传播行为是为了解决业务层方法之间互相调用的事务问题，当一个事务方法被另一个事务方法调用时，事务该以何种状态存在？

例如新方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行，等等，这些规则就涉及到事务的传播性。

关于事务的传播性，Spring 主要定义了如下几种：

```
public enum Propagation {
 REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
 SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),
 MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),
 REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),
 NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),
 NEVER(TransactionDefinition.PROPAGATION_NEVER),
 NESTED(TransactionDefinition.PROPAGATION_NESTED);
 private final int value;
 Propagation(int value) { this.value = value; }
 public int value() { return this.value; }
}
```

具体含义如下：

| 传播性           | 描述                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------ |
| REQUIRED      | 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务                                                         |
| SUPPORTS      | 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行                                                      |
| MANDATORY     | 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常                                                             |
| REQUIRES_NEW  | 创建一个新的事务，如果当前存在事务，则把当前事务挂起                                                                 |
| NOT_SUPPORTED | 以非事务方式运行，如果当前存在事务，则把当前事务挂起                                                                 |
| NEVER         | 以非事务方式运行，如果当前存在事务，则抛出异常                                                                    |
| NESTED        | 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 TransactionDefinition.PROPAGATION_REQUIRED |

一共是七种传播性，具体配置也简单：

**TransactionTemplate中的配置**

```
transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
```

**PlatformTransactionManager中的配置**

```
public void update2() {
    //创建事务的默认配置
    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    definition.setIsolationLevel(TransactionDefinition.ISOLATION_SERIALIZABLE);
    definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    TransactionStatus status = platformTransactionManager.getTransaction(definition);
    try {
        jdbcTemplate.update("update account set money = ? where username=?;", 999, "zhangsan");
        int i = 1 / 0;
        //提交事务
        platformTransactionManager.commit(status);
    } catch (DataAccessException e) {
          e.printStackTrace();
        //回滚
        platformTransactionManager.rollback(status);
    }
}
```

**声明式事务的配置（XML）**

```
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以 add 开始的方法，添加事务-->
        <tx:method name="add*"/>
        <tx:method name="insert*" isolation="SERIALIZABLE" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

**声明式事务的配置（Java）**

```
@Transactional(noRollbackFor = ArithmeticException.class,propagation = Propagation.REQUIRED)
public void update4() {
    jdbcTemplate.update("update account set money = ? where username=?;", 998, "lisi");
    int i = 1 / 0;
}
```

用就是这么来用，至于七种传播的具体含义，松哥来和大家一个一个说。

#### 5.2.1 REQUIRED

REQUIRED 表示如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

例如我有如下一段代码：

```
@Service
public class AccountService {
    @Autowired
    JdbcTemplate jdbcTemplate;
    @Transactional
    public void handle1() {
        jdbcTemplate.update("update user set money = ? where id=?;", 1, 2);
    }
}
@Service
public class AccountService2 {
    @Autowired
    JdbcTemplate jdbcTemplate;
    @Autowired
    AccountService accountService;
    public void handle2() {
        jdbcTemplate.update("update user set money = ? where username=?;", 1, "zhangsan");
        accountService.handle1();
    }
}
```

我在 handle2 方法中调用 handle1。

那么：

1. 如果 handle2 方法本身是有事务的，则 handle1 方法就会加入到 handle2 方法所在的事务中，这样两个方法将处于同一个事务中，一起成功或者一起失败（不管是 handle2 还是 handle1 谁抛异常，都会导致整体回滚）。
2. 如果 handle2 方法本身是没有事务的，则 handle1 方法就会自己开启一个新的事务，自己玩。

举一个简单的例子：handle2 方法有事务，handle1 方法也有事务（小伙伴们根据前面的讲解自行配置事务），项目打印出来的事务日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.spring_tran02.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@875256468 wrapping com.mysql.cj.jdbc.ConnectionImpl@9753d50] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@875256468 wrapping com.mysql.cj.jdbc.ConnectionImpl@9753d50] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Participating in existing transaction
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@875256468 wrapping com.mysql.cj.jdbc.ConnectionImpl@9753d50]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@875256468 wrapping com.mysql.cj.jdbc.ConnectionImpl@9753d50] after transaction
```

从日志中可以看到，前前后后一共就开启了一个事务，日志中有这么一句：

```
Participating in existing transaction
```

这个就说明 handle1 方法没有自己开启事务，而是加入到 handle2 方法的事务中了。

#### 5.2.2 REQUIRES_NEW

REQUIRES_NEW 表示创建一个新的事务，如果当前存在事务，则把**当前事务挂起**。换言之，不管外部方法是否有事务，REQUIRES_NEW 都会开启自己的事务。

这块松哥要多说两句，有的小伙伴可能觉得 REQUIRES_NEW 和 REQUIRED 太像了，似乎没啥区别。其实你要是单纯看最终回滚效果，可能确实看不到啥区别。但是，大家注意松哥上面的加粗，在 REQUIRES_NEW 中可能会同时存在两个事务，外部方法的事务被挂起，内部方法的事务独自运行，而在 REQUIRED 中则不会出现这种情况，如果内外部方法传播性都是 REQUIRED，那么最终也只是一个事务。

还是上面那个例子，假设 handle1 和 handle2 方法都有事务，handle2 方法的事务传播性是 REQUIRED，而 handle1 方法的事务传播性是 REQUIRES_NEW，那么最终打印出来的事务日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.spring_tran02.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@422278016 wrapping com.mysql.cj.jdbc.ConnectionImpl@732405c2] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@422278016 wrapping com.mysql.cj.jdbc.ConnectionImpl@732405c2] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Suspending current transaction, creating new transaction with name [org.javaboy.spring_tran02.AccountService.handle1]
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@247691344 wrapping com.mysql.cj.jdbc.ConnectionImpl@14ad4b95] for JDBC transaction
com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection com.mysql.cj.jdbc.ConnectionImpl@14ad4b95
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@247691344 wrapping com.mysql.cj.jdbc.ConnectionImpl@14ad4b95] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@247691344 wrapping com.mysql.cj.jdbc.ConnectionImpl@14ad4b95]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@247691344 wrapping com.mysql.cj.jdbc.ConnectionImpl@14ad4b95] after transaction
o.s.jdbc.support.JdbcTransactionManager  : Resuming suspended transaction after completion of inner transaction
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@422278016 wrapping com.mysql.cj.jdbc.ConnectionImpl@732405c2]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@422278016 wrapping com.mysql.cj.jdbc.ConnectionImpl@732405c2] after transaction
```

分析这段日志我们可以看到：

1. 首先为 handle2 方法开启了一个事务。
2. 执行完 handle2 方法的 SQL 之后，事务被刮起（Suspending）。
3. 为 handle1 方法开启了一个新的事务。
4. 执行 handle1 方法的 SQL。
5. 提交 handle1 方法的事务。
6. 恢复被挂起的事务（Resuming）。
7. 提交 handle2 方法的事务。

从这段日志中大家可以非常明确的看到 REQUIRES_NEW 和 REQUIRED 的区别。

松哥再来简单总结下(假设 handle1 方法的事务传播性是 REQUIRES_NEW)：

1. 如果 handle2 方法没有事务，handle1 方法自己开启一个事务自己玩。
2. 如果 handle2 方法有事务，handle1 方法还是会开启一个事务。此时，如果 handle2 发生了异常进行回滚，并不会导致 handle1 方法回滚，因为 handle1 方法是独立的事务；如果 handle1 方法发生了异常导致回滚，并且 handle1 方法的异常没有被捕获处理传到了 handle2 方法中，那么也会导致 handle2 方法回滚。

这个地方小伙伴们要稍微注意一下，我们测试的时候，由于是两个更新 SQL，如果更新的查询字段不是索引字段，那么 InnoDB 将使用表锁，这样就会发生死锁（handle2 方法执行时开启表锁，导致 handle1 方法陷入等待中，而必须 handle1 方法执行完，handle2 才能释放锁）。所以，在上面的测试中，我们要将 username 字段设置为索引字段，这样默认就使用行锁了。

#### 5.2.3 NESTED

NESTED 表示如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 TransactionDefinition.PROPAGATION_REQUIRED。

假设 handle2 方法有事务，handle1 方法也有事务且传播性为 NESTED，那么最终执行的事务日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.demo.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@2025689131 wrapping com.mysql.cj.jdbc.ConnectionImpl@2ed3628e] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@2025689131 wrapping com.mysql.cj.jdbc.ConnectionImpl@2ed3628e] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Creating nested transaction with name [org.javaboy.demo.AccountService.handle1]
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.support.JdbcTransactionManager  : Releasing transaction savepoint
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@2025689131 wrapping com.mysql.cj.jdbc.ConnectionImpl@2ed3628e]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@2025689131 wrapping com.mysql.cj.jdbc.ConnectionImpl@2ed3628e] after transaction
```

关键一句在 Creating nested transaction。

此时，NESTED 修饰的内部方法（handle1）属于外部事务的子事务，外部主事务回滚的话，子事务也会回滚，而内部子事务可以单独回滚而不影响外部主事务和其他子事务（需要处理掉内部子事务的异常）。

#### 5.2.4 MANDATORY

MANDATORY 表示如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

这个好理解，我举两个例子：

假设 handle2 方法有事务，handle1 方法也有事务且传播性为 MANDATORY，那么最终执行的事务日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.demo.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@768820610 wrapping com.mysql.cj.jdbc.ConnectionImpl@14840df2] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@768820610 wrapping com.mysql.cj.jdbc.ConnectionImpl@14840df2] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Participating in existing transaction
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@768820610 wrapping com.mysql.cj.jdbc.ConnectionImpl@14840df2]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@768820610 wrapping com.mysql.cj.jdbc.ConnectionImpl@14840df2] after transaction
```

从这段日志可以看出：

1. 首先给 handle2 方法开启事务。
2. 执行 handle2 方法的 SQL。
3. handle1 方法加入到已经存在的事务中。
4. 执行 handle1 方法的 SQL。
5. 提交事务。

假设 handle2 方法无事务，handle1 方法有事务且传播性为 MANDATORY，那么最终执行时会抛出如下异常：

```
No existing transaction found for transaction marked with propagation 'mandatory'
```

由于没有已经存在的事务，所以出错了。

#### 5.2.5 SUPPORTS

SUPPORTS 表示如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

这个也简单，举两个例子大家就明白了。

假设 handle2 方法有事务，handle1 方法也有事务且传播性为 SUPPORTS，那么最终事务执行日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.demo.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@1780573324 wrapping com.mysql.cj.jdbc.ConnectionImpl@44eafcbc] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@1780573324 wrapping com.mysql.cj.jdbc.ConnectionImpl@44eafcbc] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Participating in existing transaction
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@1780573324 wrapping com.mysql.cj.jdbc.ConnectionImpl@44eafcbc]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@1780573324 wrapping com.mysql.cj.jdbc.ConnectionImpl@44eafcbc] after transaction
```

这段日志很简单，没啥好说的，认准 Participating in existing transaction 表示加入到已经存在的事务中即可。

假设 handle2 方法无事务，handle1 方法有事务且传播性为 SUPPORTS，这个最终就不会开启事务了，也没有相关日志。

#### 5.2.6 NOT_SUPPORTED

NOT_SUPPORTED 表示以非事务方式运行，如果当前存在事务，则把当前事务挂起。

假设 handle2 方法有事务，handle1 方法也有事务且传播性为 NOT_SUPPORTED，那么最终事务执行日志如下：

```
o.s.jdbc.support.JdbcTransactionManager  : Creating new transaction with name [org.javaboy.demo.AccountService2.handle2]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
o.s.jdbc.support.JdbcTransactionManager  : Acquired Connection [HikariProxyConnection@1365886554 wrapping com.mysql.cj.jdbc.ConnectionImpl@3198938b] for JDBC transaction
o.s.jdbc.support.JdbcTransactionManager  : Switching JDBC Connection [HikariProxyConnection@1365886554 wrapping com.mysql.cj.jdbc.ConnectionImpl@3198938b] to manual commit
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where username=?;]
o.s.jdbc.support.JdbcTransactionManager  : Suspending current transaction
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL update
o.s.jdbc.core.JdbcTemplate               : Executing prepared SQL statement [update user set money = ? where id=?;]
o.s.jdbc.datasource.DataSourceUtils      : Fetching JDBC Connection from DataSource
o.s.jdbc.support.JdbcTransactionManager  : Resuming suspended transaction after completion of inner transaction
o.s.jdbc.support.JdbcTransactionManager  : Initiating transaction commit
o.s.jdbc.support.JdbcTransactionManager  : Committing JDBC transaction on Connection [HikariProxyConnection@1365886554 wrapping com.mysql.cj.jdbc.ConnectionImpl@3198938b]
o.s.jdbc.support.JdbcTransactionManager  : Releasing JDBC Connection [HikariProxyConnection@1365886554 wrapping com.mysql.cj.jdbc.ConnectionImpl@3198938b] after transaction
```

这段日志大家认准这两句就行了 ：Suspending current transaction 表示挂起当前事务；Resuming suspended transaction 表示恢复挂起的事务。

#### 5.2.7 NEVER

NEVER 表示以非事务方式运行，如果当前存在事务，则抛出异常。

假设 handle2 方法有事务，handle1 方法也有事务且传播性为 NEVER，那么最终会抛出如下异常：

```
Existing transaction found for transaction marked with propagation 'never'
```

### 5.3 回滚规则

默认情况下，事务只有遇到运行期异常（RuntimeException 的子类）以及 Error 时才会回滚，在遇到检查型（Checked Exception）异常时不会回滚。

像 1/0，空指针这些是 RuntimeException，而 IOException 则算是 Checked Exception，换言之，默认情况下，如果发生 IOException 并不会导致事务回滚。

如果我们希望发生 IOException 时也能触发事务回滚，那么可以按照如下方式配置：

Java 配置：

```
@Transactional(rollbackFor = IOException.class)
public void handle2() {
    jdbcTemplate.update("update user set money = ? where username=?;", 1, "zhangsan");
    accountService.handle1();
}
```

XML 配置：

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="m3" rollback-for="java.io.IOException"/>
    </tx:attributes>
</tx:advice>
```

另外，我们也可以指定在发生某些异常时不回滚，例如当系统抛出 ArithmeticException 异常并不要触发事务回滚，配置方式如下：

Java 配置：

```
@Transactional(noRollbackFor = ArithmeticException.class)
public void handle2() {
    jdbcTemplate.update("update user set money = ? where username=?;", 1, "zhangsan");
    accountService.handle1();
}
```

XML 配置：

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="m3" no-rollback-for="java.lang.ArithmeticException"/>
    </tx:attributes>
</tx:advice>
```

### 5.4 是否只读

只读事务一般设置在查询方法上，但不是所有的查询方法都需要只读事务，要看具体情况。

一般来说，如果这个业务方法只有一个查询 SQL，那么就没必要添加事务，强行添加最终效果适得其反。

但是如果一个业务方法中有多个查询 SQL，情况就不一样了：多个查询 SQL，默认情况下，每个查询 SQL 都会开启一个独立的事务，这样，如果有并发操作修改了数据，那么多个查询 SQL 就会查到不一样的数据。此时，如果我们开启事务，并设置为只读事务，那么多个查询 SQL 将被置于同一个事务中，多条相同的 SQL 在该事务中执行将会获取到相同的查询结果。

设置事务只读的方式如下：

Java 配置：

```
@Transactional(readOnly = true)
```

XML 配置：

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="m3" read-only="true"/>
    </tx:attributes>
</tx:advice>
```

### 5.5 超时时间

超时时间是说一个事务允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。

事务超时时间配置方式如下(单位为秒)：

Java 配置：

```
@Transactional(timeout = 10)
```

XML 配置：

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="m3" read-only="true" timeout="10"/>
    </tx:attributes>
</tx:advice>
```

在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒，默认值为-1。

## 6. 注意事项

1. **事务只能应用到 public 方法上才会有效。**

应仅将 @Transactional 注解应用于具有公开可见性的方法。如果对受 protected, private o或 package-visible 修饰的方法使用，则不会引发任何错误，但是被注解的方法不会显示已配置的事务设置。

用了，不会报错，但是不生效！

2. **事务需要从外部调用，****Spring 自调事务用会失效****。即相同类里边，A 方法没有事务，B 方法有事务，A 方法调用 B 方法，则 B 方法的事务会失效，这点尤其要注意，因为代理模式只拦截通过代理传入的外部方法调用，所以自调用事务是不生效的。**

因为代理模式只拦截通过代理传入的外部方法调用，所以自调用事务是不生效的。

3. **建议事务注解 @Transactional 一般添加在实现类上，而不要定义在接口上**，如果加在接口类或接口方法上时，只有配置基于接口的代理这个注解才会生效。

### 6.1 自调用事务失效,如何处理?

事务需要从外部调用，Spring 自调事务用会失效。即相同类里边，A 方法没有事务，B 方法有事务，A 方法调用 B 方法，则 B 方法的事务会失效，这点尤其要注意，因为代理模式只拦截通过代理传入的外部方法调用，所以自调用事务是不生效的。

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private CardService cardService;

    @Override
    public void updateUser() {
        // 一系列的逻辑
        // 超过N多行的代码
        // 一系列的逻辑
        // 事务操作更新卡金额
        try {
            cardService.updateCardAmount();
        } catch(Exception e) {
            e.printStackTrace();
            log.error("更新卡金额失败");
        }
    }

    @Transactional(rollbackFor = Exception.class)
    private void updateCardAmount() {
        // 更新卡金额
    }
}
```

**解决方案一: 外部调用**

声明一个 Service，把更新表的逻辑放过去。

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private CardService cardService;

    @Override
    public void updateUser() {
        // 一系列的逻辑
        // 超过N多行的代码
        // 一系列的逻辑
        // 事务操作更新卡金额
        try {
            cardService.updateCardAmount();
        } catch(Exception e) {
            e.printStackTrace();
            log.error("更新卡金额失败");
        }
    }
}
```

**解决方案二: 编程式事务**

```
@Service
public class UserServiceImpl implements UserService {

    @Override
    public void updateUser() {
        // 一系列的逻辑
        // 超过N多行的代码
        // 一系列的逻辑
        // 事务操作更新卡金额
        transactionTemplate.execute(new TransactionCallbackWithourResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
              try {
                    updateCardAmount();
                } catch(Exception e) {
                    e.printStackTrace();
                    log.error("更新卡金额失败");
                    status.serRollbackOnly();
                }
            }
        });
    }

    private void updateCardAmount() throws Exception {
        // 更新卡金额
    }
}
```

**解决方案3: 注解 + 自调用**

调外部方法，然后外部方法再调回来

```
@Component
public class TransactionalComponent {

    public interface Cell {
        void run() throws Exception;
    }

    @Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
    public void required(Cell cell) throws Exception {
        cell.run();
    }
}
```

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private TransactionalComponent transactionalComponent;

    @Override
    public void updateUser() {
        // 一系列的逻辑
        // 超过N多行的代码
        // 一系列的逻辑
        // 事务操作更新卡金额

        // transactionalComponent.required(()->updateCardAmount());
        // transactionalComponent.required(this::updateCardAmount());

        transactionalComponent.required(new TransactionalComponent() {
            @Override
            public void run() throws Exception {
                updateCardAmount();
            }
        });
    }

    private void updateCardAmount() throws Exception {
        // 更新卡金额
    }
}
```

优化以上代码:

使用静态方法调用，不用每次都用 @Autowired 注入一次。

```
@Component
public class TransactionalComponent {

    public interface Cell {

        void run() throws Exception;
    }

    @Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
    public void required(Cell cell) throws Exception {

        cell.run();
    }
}
```

```
public class TransactionalUtils {

    private static volatile TransactionalComponent transactionalComponent;

    private static synchronized TransactionalComponent getTransactionalComponent() {
        if (transactionalComponent == null) {
            // 从容器中获取 transactionalComponent
            transactionalComponent = ApplicationContextUtils.getBean(TransactionalComponent.class);
        }
        return transactionalComponent;
    }

    public static void required(TransactionalComponent.Cell cell) throws Exception {
        getTransactionalComponent().required(cell);
    }

}
```

```
@Service
public class UserServiceImpl implements UserService {

    @Override
    public void updateUser() {
        // 一系列的逻辑
        // 超过N多行的代码
        // 一系列的逻辑
        // 事务操作更新卡金额
        TransactionalUtils.required(this::updateCardAmount());
    }

    private void updateCardAmount() throws Exception {
        // 更新卡金额
    }
}
```

原文链接: [长文捋明白 Spring 事务！隔离性？传播性？一网打尽！](https://mp.weixin.qq.com/s/6tRPXwXnWUW4mVfCdBlkog)

参考链接:

[java - Spring 自调用事务失效，你是怎么解决的？ - 小航的技术笔记 - SegmentFault 思否](https://segmentfault.com/a/1190000037765958)
