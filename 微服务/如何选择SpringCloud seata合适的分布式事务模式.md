# 如何选择SpringCloud seata合适的分布式事务模式

## Seata 分布式事务原理

Seata 是一种分布式事务中间件，旨在通过协调微服务来保持跨多个数据库的事务一致性。它的工作原理主要基于以下几个组件：

### 1. 事务协调器 (TC)

事务协调器是 Seata 的核心，负责维护全局事务的状态。它处理事务的开始、提交或回滚。

### 2. 事务管理器 (TM)

事务管理器负责定义全局事务的范围。它会向 TC 注册全局事务，并根据业务逻辑需要请求提交或回滚事务。

### 3. 资源管理器 (RM)

资源管理器负责管理微服务所访问的本地资源（如数据库）。它会向 TC 注册资源，并报告本地事务的成功或失败状态。

### 4. 数据一致性

为了保证数据一致性，Seata 实现了几种数据一致性策略，包括 AT、TCC、Saga 和 XA。

- **AT (Automatic Transaction) 模式**：二阶段提交，自动化的本地事务代理方式，适用于简单的 CRUD 操作。
- **TCC (Try-Confirm-Cancel) 模式**：三阶段提交，适用于执行时间较长或需要手动干预的业务处理。
- **Saga 模式**：适用于长事务处理，通过定义一系列本地事务来管理。
- **XA 模式**：基于 XA 协议的两阶段提交，适用于严格一致性需求的场景。

## 业务场景与分布式事务模式选择

在选择分布式事务模式时，需要考虑业务的特性和技术环境。以下是一些常见的业务场景及其建议的事务模式：

### 场景一：简单的跨服务数据更新

- **推荐模式**：AT 模式
- **理由**：AT 模式通过代理数据源的方式自动管理分布式事务，无需额外编码，易于实现。

#### 场景描述

假设有一个电商系统，用户下单时需要同时更新库存服务和订单服务。

#### 实现步骤

1. **配置 Seata Server**：首先确保 Seata Server 正在运行，并且各个微服务都配置有正确的 Seata 数据源代理。
2. **服务集成**：在库存服务和订单服务中引入 Seata 相关依赖，并配置好数据源代理。
3. 业务代码实现：
   - 在订单服务中，创建订单并调用库存服务的 API 减少库存。

配置 Seata

```xml
<dependencies>
    <!-- Seata Starter -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-spring-boot-starter</artifactId>
        <version>${seata.version}</version>
    </dependency>

    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Boot Starter JDBC -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <!-- 数据库连接池 -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
    </dependency>

    <!-- 数据库驱动，以MySQL为例 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>

```

在 `application.properties` 文件中为库存服务和订单服务配置 Seata：

```properties
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/inventory_db?useSSL=false
    username: user
    password: password
    hikari:
      maximum-pool-size: 20

  cloud:
    alibaba:
      seata:
        enabled: true
        tx-service-group: my_tx_group
        application-id: inventory-service
        seata:
          service:
            vgroup-mapping:
              my_tx_group: default
            grouplist:
              default: 127.0.0.1:8091
          config:
            type: file

seata:
  enabled: true                    # 启用seata配置
  application-id: inventory-service # 应用标识
  tx-service-group: my_tx_group  # 事务服务组
  service:
    vgroup-mapping:    # 事务组和事务协调器（TC）服务组的映射，允许你指定哪个事务组应该使用哪个 TC 实例
      my_tx_group: default  
    grouplist:    # 指定 TC 地址,这里使用的是本地地址和默认端口
      default: 127.0.0.1:8091
  config:  #  Seata 配置类型，这里使用的是文件类型（file），也可以配置为中心化配置服务（如 nacos, consul等）
    type: file
  registry:  # 注册中心类型，这里同样使用文件。对于生产环境，可能会使用 nacos、eureka 或其他支持的注册中心。
    type: eureka

```

配置 Seata DataSourceProxy

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        HikariDataSource dataSource = properties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
        return new DataSourceProxy(dataSource);
    }
}

```

这个配置确保所有数据库操作都通过 Seata 的 `DataSourceProxy` 进行，从而允许 Seata 追踪并管理这些操作，以保证在发生失败时可以正确回滚。

Seata 就能通过代理数据源管理事务了，确保在发生异常时能够正确地回滚事务，从而维护数据的一致性。(**Seata 通过代理数据源拦截所有的数据库操作**，确保在发生错误时可以回滚所有涉及的服务，包括被调用方的数据库更改。**不仅是调用方，被调用方服务也需要配置代理数据源**。这是因为 Seata 需要能够管理和协调所有参与事务的微服务的本地事务，确保事务的原子性和一致性)

库存服务

```java
// 库存服务 - InventoryController.java
@RestController
@RequestMapping("/inventory")
public class InventoryController {

    @Autowired
    private InventoryService inventoryService;

    @PostMapping("/reduce")
    public ResponseEntity<String> reduceInventory(@RequestBody InventoryReduction reduction) {
        boolean result = inventoryService.reduceStock(reduction.getProductId(), reduction.getQuantity());
        if (result) {
            return ResponseEntity.ok("Inventory reduced successfully.");
        } else {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Failed to reduce inventory.");
        }
    }
}

// 库库存服务 - InventoryService.java
@Service
public class InventoryService {

    @Autowired
    private InventoryRepository inventoryRepository;

    @Transactional
    public boolean reduceStock(Long productId, Integer quantity) {
        // 逻辑: 检查库存，足够则减少相应数量
        Inventory inventory = inventoryRepository.findByProductId(productId);
        if (inventory != null && inventory.getQuantity() >= quantity) {
            inventory.setQuantity(inventory.getQuantity() - quantity);
            inventoryRepository.save(inventory);
            return true;
        }
        return false;
    }
}

```

订单服务

```java
// 订单服务 - OrderController.java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping("/create")
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        Order createdOrder = orderService.createOrder(order);
        return ResponseEntity.ok(createdOrder);
    }
}

// 订单服务 - OrderService.java
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private RestTemplate restTemplate;

    @Transactional
    public Order createOrder(Order order) {
        // 先尝试减少库存
        HttpEntity<InventoryReduction> request = new HttpEntity<>(new InventoryReduction(order.getProductId(), order.getQuantity()));
        ResponseEntity<String> response = restTemplate.postForEntity("http://inventory-service/inventory/reduce", request, String.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            // 库存减少成功后，创建订单
            return orderRepository.save(order);
        } else {
            throw new RuntimeException("Failed to reduce inventory, order not created.");
        }
    }
}

```

### 场景二：需要复杂业务处理的场景

- **推荐模式**：TCC 模式
- **理由**：TCC 模式允许自定义业务操作的各个阶段，适合业务逻辑复杂且需要明确事务状态的场景。

#### 场景描述

假设有一个简单的金融系统，用户在进行资金转账时涉及两个操作：从一个账户扣款（debit）和向另一个账户存款（credit）。在这个场景中，我们将使用 Seata 的 TCC 模式来保证转账操作的一致性。

用户需要从账户 A 转移资金到账户 B。这个操作需要两个步骤：首先从账户 A 扣除相应金额，然后向账户 B 增加同等金额。如果任何一个步骤失败，整个操作都应该回滚。

#### 实现步骤

1. 添加 Seata TCC 相关依赖

首先，确保项目中包含了 Seata TCC 的依赖。这通常包括 Seata 的核心库和任何需要的 Spring Cloud 配件。

```xml
<dependencies>
    <!-- Seata Starter -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-spring-boot-starter</artifactId>
        <version>{seata-version}</version>
    </dependency>
    <!-- Spring Boot Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Boot Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- MySQL Driver -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>

```

2. 定义 TCC 接口

在服务中定义涉及 TCC 模式的接口。这里我们需要实现 Try, Confirm 和 Cancel 方法。

```java
public interface AccountService {

    // Try 阶段：尝试扣款
    boolean prepareDebit(Long accountId, Double amount);

    // Confirm 阶段：确认扣款
    boolean confirmDebit(Long accountId, Double amount);

    // Cancel 阶段：取消扣款
    boolean cancelDebit(Long accountId, Double amount);

    // Try 阶段：尝试存款
    boolean prepareCredit(Long accountId, Double amount);

    // Confirm 阶段：确认存款
    boolean confirmCredit(Long accountId, Double amount);

    // Cancel 阶段：取消存款
    boolean cancelCredit(Long accountId, Double amount);
}

```

3. 实现业务逻辑

对于每一个阶段（Try, Confirm, Cancel），我们需要实现相应的业务逻辑。

```java
@Service
@Transactional
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountRepository accountRepository;

    @Override
    public boolean prepareDebit(Long accountId, Double amount) {
        // 锁定账户并检查资金
        Account account = accountRepository.findById(accountId).orElse(null);
        if (account != null && account.getBalance() >= amount) {
            account.setReserved(account.getReserved() + amount);
            accountRepository.save(account);
            return true;
        }
        return false;
    }

    @Override
    public boolean confirmDebit(Long accountId, Double amount) {
        Account account = accountRepository.findById(accountId).orElse(null);
        if (account != null) {
            account.setBalance(account.getBalance() - amount);
            account.setReserved(account.getReserved() - amount);
            accountRepository.save(account);
            return true;
        }
        return false;
    }

    @Override
    public boolean cancelDebit(Long accountId, Double amount) {
        Account account = accountRepository.findById(accountId).orElse(null);
        if (account != null) {
            account.setReserved(account.getReserved() - amount);
            accountRepository.save(account);
            return true;
        }
        return false;
    }

    @Override
    public boolean prepareCredit(Long accountId, Double amount) {
        // 此阶段通常不需要操作，返回true表示可以进入下一步
        return true;
    }

    @Override
    public boolean confirmCredit(Long accountId, Double amount) {
        Account account = accountRepository.findById(accountId).orElse(null);
        if (account != null) {
            account.setBalance(account.getBalance() + amount);
            accountRepository.save(account);
            return true;
        }
        return false;
    }

    @Override
    public boolean cancelCredit(Long accountId, Double amount) {
        // 存款操作的Cancel通常不执行操作，因为Prepare阶段没有实际操作
        return true;
    }
}

```

4. 配置和使用 Seata TCC

在您的 `application.yml` 中，确保配置了 Seata TCC 事务组，并且您的服务正确地注册到 Seata Server。在服务调用端，您需要确保调用这些方法时遵循 TCC 模式的要求，使用 Seata 的 TCC API 来正确开始事务、调用 Try 阶段的方法，以及根据结果来提交或回滚事务。

```yaml
spring:
  application:
    name: transfer-service
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/transfer_db?useSSL=false&serverTimezone=UTC
    username: dbuser
    password: dbpassword
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

  cloud:
    alibaba:
      seata:
        tx-service-group: my_tx_group

seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: ${spring.cloud.alibaba.seata.tx-service-group}
  service:
    vgroup-mapping:
      my_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
  config:
    type: file
  registry:
    type: file

```

5. 继续配置和使用 Seata TCC

在服务调用端，确保在启动类中添加 Seata TCC 的事务管理器配置。此外，需要在服务间调用时正确处理事务的边界，确保全局事务的一致性。

```java
@Configuration
public class SeataConfig {
    @Bean
    public GlobalTransactionScanner globalTransactionScanner() {
        return new GlobalTransactionScanner("transfer-service", "my_tx_group");
    }
}

```

在使用 Seata 的 AT 模式或 TCC 模式进行分布式事务处理时，所有参与事务的服务（无论是调用方还是被调用方）都需要配置代理数据库。

6. 调用 TCC 方法

在业务流程中，需要编写代码调用 Try 方法，并根据其结果决定是否执行 Confirm 或 Cancel。这通常在一个业务服务层实现，如下所示：

```java
@Service
public class TransferService {

    @Autowired
    private AccountService accountService;

    public String transfer(Long fromAccountId, Long toAccountId, Double amount) {
        boolean debitPrepared = false;
        boolean creditPrepared = false;

        try {
            // Try 阶段
            debitPrepared = accountService.prepareDebit(fromAccountId, amount);
            if (!debitPrepared) {
                return "Insufficient funds to debit.";
            }
            
            creditPrepared = accountService.prepareCredit(toAccountId, amount);
            if (!creditPrepared) {
                // Credit preparation failed, trigger debit cancel
                accountService.cancelDebit(fromAccountId, amount);
                return "Failed to prepare credit.";
            }

            // Confirm 阶段
            accountService.confirmDebit(fromAccountId, amount);
            accountService.confirmCredit(toAccountId, amount);

            return "Transfer successful";
        } catch (Exception e) {
            // Exception handling
            if (debitPrepared) {
                accountService.cancelDebit(fromAccountId, amount);
            }
            if (creditPrepared) {
                accountService.cancelCredit(toAccountId, amount);
            }
            return "Transfer failed due to an exception: " + e.getMessage();
        }
    }
}

```

在这个 TCC 模式的示例中，我们展示了如何在涉及资金转移的业务场景中实现和维护数据一致性。通过分别定义 Try, Confirm, 和 Cancel 操作，并在服务调用中管理全局事务的生命周期，Seata 确保了操作的一致性和原子性。



### 场景三：长时间运行的业务操作

- **推荐模式**：Saga 模式
- **理由**：Saga模式通常适用于长时间运行的业务流程，需要通过一系列的本地事务来维持整体的一致性。Saga 模式通过分解业务操作为一系列可补偿的步骤（本地事务），每步执行成功后，才继续下一步。如果任何步骤失败，之前的步骤将通过定义的补偿操作（回滚逻辑）来撤销。

#### 场景描述

假设有一个旅行预订系统，用户在预订旅行时需要依次预订机票、酒店和租车服务。如果在预订过程中任何一步失败，之前的所有预订都需要取消，每项服务都可能耗时较长。

#### 实现步骤

1. **定义各步骤的本地事务**：每个服务需要定义相应的本地事务。
2. **配置 Saga 流程**：在 Seata 中定义整个旅行预订的 Saga 流程，包括各个服务的调用关系。

首先, 配置基本的 Seata 配置,包括应用标识、Seata 服务组、配置类型以及注册中心类型等;

对于 Saga 模式，数据源配置不需要使用 Seata 的 `DataSourceProxy`（这是针对 AT 模式的配置）。但是，你仍然需要正确配置数据源以支持正常的数据库操作：

```yaml
spring:
  application:
    name: travel-service
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/travel_db?useSSL=false&serverTimezone=UTC
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: my_tx_group
  service:
    vgroup-mapping:
      my_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
  config:
    type: file
  registry:
    type: file
  saga:
    json-parser: fastjson
    auto-register-resources: true
    resources:
      - classpath:saga/travel-saga.json

logging:
  level:
    io.seata: INFO

```

> - **spring.application.name**: 设置 Spring Boot 应用的名称，这个名称将被用作 Seata 的 `application-id`。
> - **spring.datasource**: 配置数据库连接，包括驱动、URL、用户名和密码。
> - **spring.jpa**: 设置 JPA 的一些参数，如自动创建/更新表结构和是否显示 SQL 日志。
> - **seata.enabled**: 启用 Seata 集成。
> - **seata.application-id** 和 **seata.tx-service-group**: 配置应用的 ID 和事务服务组，这些是在 Seata 中注册服务时使用的。
> - **seata.service.vgroup-mapping** 和 **seata.service.grouplist**: 配置事务服务组和 Seata 服务器的映射。
> - **seata.config.type** 和 **seata.registry.type**: 配置 Seata 的配置源和注册中心类型，这里使用的是本地文件。
> - **seata.saga.json-parser**: 指定 Saga JSON 文件的解析器。
> - **seata.saga.auto-register-resources**: 设置是否自动注册 Saga 资源。
> - **seata.saga.resources**: 指定 Saga JSON 文件的路径。
> - **logging.level.io.seata**: 设置 Seata 相关日志的输出级别。

其次，我们需要定义用于预订和取消预订的服务接口。

```java
public interface TravelService {
    // 预定机票
    boolean bookFlight(Long userId, FlightBookingDetails details);
    // 取消机票
    boolean cancelFlight(Long userId, FlightBookingDetails details);
    
    // 预定旅馆
    boolean bookHotel(Long userId, HotelBookingDetails details);
    // 取消旅馆
    boolean cancelHotel(Long userId, HotelBookingDetails details);

    // 预定租车
    boolean bookCar(Long userId, CarBookingDetails details);
    // 取消租车
    boolean cancelCar(Long userId, CarBookingDetails details);
}

```

接下来，我们为每项服务提供实现。这里只展示机票预订的实现作为示例：

```java
@Service
public class TravelServiceImpl implements TravelService {
    @Autowired
    private FlightService flightService;
    @Autowired
    private HotelService hotelService;
    @Autowired
    private CarService carService;

    @Override
    public boolean bookFlight(Long userId, FlightBookingDetails details) {
        return flightService.bookFlight(details);
    }

    @Override
    public boolean cancelFlight(Long userId, FlightBookingDetails details) {
        return flightService.cancelFlight(details);
    }

    @Override
    public boolean bookHotel(Long userId, HotelBookingDetails details) {
        return hotelService.bookHotel(details);
    }

    @Override
    public boolean cancelHotel(Long userId, HotelBookingDetails details) {
        return hotelService.cancelHotel(details);
    }

    @Override
    public boolean bookCar(Long userId, CarBookingDetails details) {
        return carService.bookCar(details);
    }

    @Override
    public boolean cancelCar(Long userId, CarBookingDetails details) {
        return carService.cancelCar(details);
    }
}

```

配置`travel-saga.json`，在Seata中，可以通过定义一个JSON格式的Saga状态机来实现Saga编排。以下是一个简单的示例：

```json
{
  "Name": "TravelBookingSaga",
  "StartState": "BookFlight",
  "States": [
    {
      "Name": "BookFlight",
      "Type": "Task",
      "ResourceName": "flightService.bookFlight",
      "NextState": "BookHotel",
      "FailState": "CompensateFlight",
      "Description": "开始 Saga 流程，尝试预订机票。成功则进入 BookHotel；失败则进入 CompensateFlight 进行回滚。"
    },
    {
      "Name": "BookHotel",
      "Type": "Task",
      "ResourceName": "hotelService.bookHotel",
      "NextState": "BookCar",
      "FailState": "CompensateHotel",
      "Description": "尝试预订酒店。成功则进入 BookCar；失败则触发 CompensateHotel 开始之前步骤的回滚。"
    },
    {
      "Name": "BookCar",
      "Type": "Task",
      "ResourceName": "carService.bookCar",
      "EndState": true,
      "FailState": "CompensateCar",
      "Description": "预订序列中的最后一个任务，尝试预订汽车。成功则结束 Saga；失败则从 CompensateCar 开始补偿。"
    },
    {
      "Name": "CompensateFlight",
      "Type": "Compensation",
      "ResourceName": "flightService.cancelFlight",
      "EndState": true,
      "Description": "通过取消机票来补偿机票预订。这是机票补偿序列的结束状态。"
    },
    {
      "Name": "CompensateHotel",
      "Type": "Compensation",
      "ResourceName": "hotelService.cancelHotel",
      "NextState": "CompensateFlight",
      "Description": "回滚酒店预订。执行后，继续执行 CompensateFlight 以继续补偿过程。"
    },
    {
      "Name": "CompensateCar",
      "Type": "Compensation",
      "ResourceName": "carService.cancelCar",
      "NextState": "CompensateHotel",
      "Description": "开始通过取消汽车预订的回滚过程。完成后，进入 CompensateHotel 继续解除预订序列。"
    }
  ]
}

```

> - **ResourceName**：这应该是能够触发特定服务操作的标识符。通常，这会是一个具体的服务方法名或者一个通过 HTTP 调用的端点。
> - **Type**：任务类型，通常是 "Task" 或 "Compensation"。
> - **NextState** 和 **FailState**：分别指明在当前任务成功或失败后应转向的状态。
>
> 每个 `ResourceName` 应对应一个在服务中实现的操作。例如，如果你使用 Spring Boot 构建服务，你可能会在服务中定义相应的 REST 控制器或服务方法：
>
> ```java
> @RestController
> public class FlightBookingController {
>     @PostMapping("/bookFlight")
>     public ResponseEntity<?> bookFlight(@RequestBody FlightBookingDetails details) {
>         // 业务逻辑
>         return ResponseEntity.ok().build();
>     }
> 
>     @PostMapping("/cancelFlight")
>     public ResponseEntity<?> cancelFlight(@RequestBody FlightBookingDetails details) {
>         // 补偿逻辑
>         return ResponseEntity.ok().build();
>     }
> }
> ```

### 场景四：对数据一致性要求极高的业务

- **推荐模式**：XA 模式
- **理由**：XA 模式基于标准的两阶段提交协议，虽然性能成本较高，但可以提供最严格的数据一致性保证。

**XA 协议通过二阶段提交协议保证强一致性。**

二阶段提交协议，顾名思义，它具有两个阶段：**第一阶段准备，第二阶段提交**。这里，事务管理者（协调者）主要负责控制所有节点的操作结果，包括准备流程和提交流程。

第一阶段，事务管理者（协调者）向资源管理者（参与者）发起准备指令，询问资源管理者（参与者）预提交是否成功。如果资源管理者（参与者）可以完成，就会执行操作，并不提交，最后给出自己响应结果，是预提交成功还是预提交失败。

第二阶段，如果全部资源管理者（参与者）都回复预提交成功，资源管理者（参与者）正式提交命令。如果其中有一个资源管理者（参与者）回复预提交失败，则事务管理者（协调者）向所有的资源管理者（参与者）发起回滚命令。

举个案例，现在我们有一个事务管理者（协调者），三个资源管理者（参与者），那么这个事务中我们需要保证这三个参与者在事务过程中的数据的强一致性。首先，事务管理者（协调者）发起准备指令预判它们是否已经预提交成功了，如果全部回复预提交成功，那么事务管理者（协调者）正式发起提交命令执行数据的变更。

> 注意的是，虽然二阶段提交协议为保证强一致性提出了一套解决方案，但是仍然存在一些问题。
>
> 其一，事务管理者（协调者）主要负责控制所有节点的操作结果，包括准备流程和提交流程，但是整个流程是同步的，所以事务管理者（协调者）必须等待每一个资源管理者（参与者）返回操作结果后才能进行下一步操作。这样就非常容易造成**同步阻塞**问题。
>
> 其二，**单点故障**也是需要认真考虑的问题。事务管理者（协调者）和资源管理者（参与者）都可能出现宕机，如果资源管理者（参与者）出现故障则无法响应而一直等待，事务管理者（协调者）出现故障则事务流程就失去了控制者，换句话说，就是整个流程会一直阻塞，甚至极端的情况下，一部分资源管理者（参与者）数据执行提交，一部分没有执行提交，也会出现数据不一致性。

#### 场景描述

在银行系统中，跨国货币交换需要极高的数据一致性。

#### 实现步骤

1. **配置 XA 数据源**：为参与 XA 事务的服务配置支持 XA 的数据源。
2. **业务代码实现**：在业务服务中使用 XA 事务管理器。

1. 添加依赖

首先，确保在你的 `pom.xml` 或 `build.gradle` 文件中包含了 Seata 和数据库驱动的依赖：

```xml
<dependencies>
    <!-- Seata -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-spring-boot-starter</artifactId>
        <version>1.4.2</version>  <!-- 使用适当的版本 -->
    </dependency>
    
    <!-- MySQL XA 数据源 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.23</version> <!-- 使用适当的版本 -->
    </dependency>

    <!-- JTA 事务管理器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jta-atomikos</artifactId>
    </dependency>
</dependencies>

```

2. 配置 application.yml

在 `application.yml` 中配置 Seata 以及数据库和 JTA 事务管理器。以下是基础配置，可以根据实际需要进行调整：

```yaml
spring:
  application:
    name: xa-demo-application
  datasource:
    # Atomikos JTA 数据源配置
    jta:
      atomikos:
        datasource:
          unique-resource-name: dataSource
          xa-data-source-classname: com.mysql.cj.jdbc.MysqlXADataSource
          xa-properties:
            serverName: localhost
            portNumber: 3306
            databaseName: your_database
            user: your_username
            password: your_password

seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: my_tx_group
  service:
    vgroup-mapping:
      my_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
  config:
    type: file
  registry:
    type: file

```

3. 配置 Seata XA 数据源代理

在 Spring Boot 配置类中创建 Seata XA 数据源代理，确保全局事务的管理：

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        MysqlXADataSource mysqlXADataSource = new MysqlXADataSource();
        mysqlXADataSource.setUrl("jdbc:mysql://localhost:3306/your_database");
        mysqlXADataSource.setUser("your_username");
        mysqlXADataSource.setPassword("your_password");

        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXADataSource);
        xaDataSource.setUniqueResourceName("xaDataSource");

        return new DataSourceProxyXA(xaDataSource);
    }
}

```

现在，你的应用配置了支持 XA 事务的数据源。你可以在你的业务服务中使用 `@Transactional` 注解来开启和管理事务。确保在业务逻辑中适当地处理异常，以便在出现错误时可以触发全局回滚。

> 被调用方的代理数据源需要与调用方的代理数据源保持一致。在分布式事务中，所有涉及到的数据库操作都需要通过同一个事务管理器来进行协调，以保证全局事务的一致性和可靠性。因此，所有服务都应该使用相同的事务管理器和相同的代理数据源，这样才能确保不同服务之间的事务操作能够正确地协调和提交或回滚。

4. 使用 @Transactional 管理事务

在业务逻辑中使用 Spring 的 `@Transactional` 注解来管理事务。确保在需要事务控制的服务或方法上加上此注解：

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {
        // 业务逻辑，可能涉及多个数据库操作
    }
}

```



通过以上步骤，你可以在你的 Spring Boot 应用中实现 Seata 的 XA 模式，从而在多个数据源间保持强一致性。这种方法适用于需要严格事务

## 在配置 AT 模式和 TCC 模式时，seata代理数据源为什么不同?

### AT 模式配置

在 **AT (Automatic Transaction) 模式**下，主要依赖于 Seata 的 `DataSourceProxy` 来拦截和管理数据库操作。这是确保在分布式事务中数据一致性的关键。

#### 配置 DataSourceProxy

对于使用 AT 模式的服务，需要配置 `DataSourceProxy` 来管理数据库连接。以下是一个典型的配置示例：

```java
javaCopy code@Configuration
public class DataSourceProxyConfig {

    @Bean
    @Primary
    public DataSource dataSource(DataSourceProperties dataSourceProperties) {
        HikariDataSource dataSource = dataSourceProperties.initializeDataSourceBuilder()
                                        .type(HikariDataSource.class).build();
        return new DataSourceProxy(dataSource);
    }
}
```

这个配置确保所有数据库操作都通过 Seata 的 `DataSourceProxy` 进行，从而允许 Seata 追踪并管理这些操作，以保证在发生失败时可以正确回滚。

### TCC 模式配置

而在 **TCC (Try-Confirm-Cancel) 模式**下，关注的重点是如何定义和管理每个业务操作的三个阶段：Try、Confirm 和 Cancel。这通常涉及到编写具体的业务逻辑，并通过 Seata 的 `GlobalTransactionScanner` 来扫描和管理这些 TCC 方法。

#### 配置 GlobalTransactionScanner

`GlobalTransactionScanner` 是 Seata 在 Spring 环境中用于自动注册 TCC 事务和管理分布式事务生命周期的组件。这通常在 Spring Boot 的配置类中进行配置：

```java
@Configuration
public class SeataConfig {
    @Bean
    public GlobalTransactionScanner globalTransactionScanner() {
        // "my-service-name" 应为服务名，"my_tx_group" 应为事务组名，与 application.yml 中配置一致
        return new GlobalTransactionScanner("my-service-name", "my_tx_group");
    }
}
```

这个配置的作用是启用 Seata 的全局事务管理功能，自动扫描带有 `@GlobalTransactional` 注解的方法，处理 TCC 事务的三个阶段。

### 结论

在配置 AT 模式和 TCC 模式时，核心差异在于：

- **AT 模式** 需要配置 `DataSourceProxy`，主要用于自动管理普通的 JDBC 事务，使其适用于分布式环境。
- **TCC 模式** 需要配置 `GlobalTransactionScanner`，这主要用于管理复杂的业务逻辑，涉及显式定义的三个阶段（Try, Confirm, Cancel）,通过注解注入。

两种模式都需要在 `application.yml` 中配置 Seata 服务的通用属性，如服务组和事务组信息。

## 为什么TCC模式使用@GlobalTransactional,但XA模式使用@Transactional

`@Transactional` 和 `@GlobalTransactional` 注解在 Seata 中的使用是基于不同的事务模式和设计理念。

1. **`@Transactional` 注解**：
   - `@Transactional` 注解通常用于传统的本地事务管理，例如在单个数据库上执行的事务。当你在 Spring Boot 应用中使用 `@Transactional` 注解时，Spring 框架会在方法或类级别上开启事务，并将其管理为本地事务。
   - 在 Seata 的 XA 模式中，虽然也使用了分布式事务的协调和管理，但是底层的事务实现是基于 XA 协议的全局事务。因此，虽然你使用了 `@Transactional` 注解来标记方法，但实际上这些方法会被 Seata 的事务代理拦截，并转换为分布式事务的一部分，而不是简单的本地事务。
2. **`@GlobalTransactional` 注解**：
   - `@GlobalTransactional` 注解是 Seata 提供的针对分布式事务的全局事务管理器的一部分。它用于标记一个全局事务的边界，可以跨越多个微服务。当你在方法上标记了 `@GlobalTransactional` 注解时，Seata 会为该方法开启一个全局事务，并在其中协调各个微服务的本地事务。
   - 在 Seata 的 TCC 模式中，`@GlobalTransactional` 注解常用于标记 TCC 操作的入口方法。TCC 模式要求在事务的不同阶段执行不同的逻辑（Try、Confirm、Cancel），而 `@GlobalTransactional` 注解可以确保这些逻辑在全局事务的范围内得到正确执行，从而保证了整个事务的一致性和可靠性。

因此，尽管在 Seata 的 XA 模式和 TCC 模式中都需要使用分布式事务，但是它们基于不同的事务模型和设计理念，因此在标记事务边界和管理事务的时候使用了不同的注解。

## 总结

选择合适的分布式事务模式是确保微服务架构下数据一致性的关键。通过了解不同模式的特点和适用场景，可以更好地为具体业务需求设计合适的事务管理策略。Seata 作为一种灵活的分布式事务解