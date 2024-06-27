# Springboot 配置多数据源

## 基于Mybatis

目录结构

```
- com.sangto.task                    
    - cofnig                    // 配置类
    - dao                       // dao层
        - read                  // 读取dao
        - write                 // 写入dao
    - domain                    // domain
    - service                   // service接口
        - impl                  // service实现类
    MainApplication.class       // 启动类


- resource
    - mapper                    // mapper文件
        - read                  // 读取mapper
        - write                 // 写入mapper
    - application.yml           // 配置文件
```

### Springboot 引入Mybatis

```xml
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <druid.version>1.1.9</druid.version>
        <mybatis.version>2.2.0</mybatis.version>
        <commons-lang3.version>3.4</commons-lang3.version>
        <druid.version>1.1.9</druid.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${commons-lang3.version}</version>
        </dependency>

        <!-- alibaba的druid数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <!-- 扫描src/main/java下所有xx.xml文件 -->
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/read/*.xml</include>
                    <include>**/write/*.xml</include>
                </includes>
            </resource>
            <!-- 扫描resources下所有资源 -->
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
```

application.yml

```yaml
server:
  port: 9999
spring:
  application:
    name: sangto_task_sync
  datasource:
    # 读数据库
    read:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/test01?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
      username: root
      password: root
      type: com.alibaba.druid.pool.DruidDataSource
    # 写数据库
    write:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
      username: root
      password: root
      type: com.alibaba.druid.pool.DruidDataSource

mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

将`yaml`配置文件中的数据连接加入Springioc容器,新增配置类

`ReadMybatisConfig.java` 和 `WriteMybatisConfig.java`

```java
package com.sangto.task.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;

/**
 * @author: pangdi
 * @Date: 2023/4/25 18:29
 * @Description: 读取mybatis配置类
 */
@Configuration
@MapperScan(basePackages = "com.sangto.task.dao.read",sqlSessionFactoryRef = "readSqlSessionFactory")
public class ReadMybatisConfig {

    @Primary
    @Bean(name = "readDataSource") 
    // 与yaml文件中路由保持一致
    @ConfigurationProperties("spring.datasource.read")
    public DataSource masterDataSource(){
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "readSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("readDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath:mapper/read/*.xml"));
        return sessionFactoryBean.getObject();
    }
}
```

```java
package com.sangto.task.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;

/**
 * @author: pangdi
 * @Date: 2023/4/25 18:29
 * @Description: 写入mybatis配置类
 */
@Configuration
@MapperScan(basePackages = "com.sangto.task.dao.write",sqlSessionFactoryRef = "writeSqlSessionFactory")
public class WriteMybatisConfig {

    @Primary
    @Bean(name = "writeDataSource")
    @ConfigurationProperties("spring.datasource.write")
    public DataSource masterDataSource(){
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "writeSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("writeDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath:mapper/write/*.xml"));
        return sessionFactoryBean.getObject();
    }
}
```

**由于我们配置多数据源,因此在启动类时,需要移除默认数据源,让springboot加载我们的配置类`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`**

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@MapperScan(basePackages = "com.sangto.board.dao")
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication .class, args);
    }
}

```

 service接口编写测试

```java
@Service
public class TaskServiceImpl implements TaskService {
    @Autowired
    private WriteMapper writeMapper;
    @Autowired
    private ReadMapper readMapper;

    /**
     * 读出然后写入
     */
    @Override
    public void taskReadToWrite() { 
        List<ReadUser> readUsers = readMapper.selectUsers();
        readUsers.forEach(e -> writeMapper.insert(e));
    }
}
```

## 利用JdbcTemplate 构建数据源

Springboot 引入Mybatis

```xml
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <druid.version>1.1.9</druid.version>
        <commons-lang3.version>3.4</commons-lang3.version>
        <druid.version>1.1.9</druid.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${commons-lang3.version}</version>
        </dependency>

        <!-- alibaba的druid数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <!-- 扫描src/main/java下所有xx.xml文件 -->
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/read/*.xml</include>
                    <include>**/write/*.xml</include>
                </includes>
            </resource>
            <!-- 扫描resources下所有资源 -->
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
```

application.yml

```yaml
server:
  port: 9999
spring:
  application:
    name: sangto_task_sync
  datasource:
    # 读数据库
    read:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/test01?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
      username: root
      password: root
      type: com.alibaba.druid.pool.DruidDataSource
    # 写数据库
    write:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
      username: root
      password: root
      type: com.alibaba.druid.pool.DruidDataSource

mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

将`yaml`配置文件中的数据连接加入Springioc容器,新增配置类`DataSourceConfig.java`

```java
package com.sangto.board.config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;

/**
 * @author: pangdi
 * @Date: 2023/4/25 18:29
 * @Description: 数据源配置类，用于创建读写两个数据源及对应的JdbcTemplate对象。
 */
@Configuration
public class DataSourceConfig {

    /**
     * 创建用于读取数据的数据源对象。
     *
     * @return 读取数据源对象
     */
    @Primary
    @Bean(name = "readDataSource")
    @ConfigurationProperties("spring.datasource.read")
    public DataSource readDataSource(){
        return DataSourceBuilder.create().build();
    }

    /**
     * 创建用于读取数据的JdbcTemplate对象。
     *
     * @param dataSource 读取数据源对象
     * @return 读取JdbcTemplate对象
     */
    @Bean(name = "readJdbcTemplate")
    public JdbcTemplate readJdbcTemplate(@Qualifier("readDataSource")DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    /**
     * 创建用于写入数据的数据源对象。
     *
     * @return 写入数据源对象
     */
    @Primary
    @Bean(name = "writeDataSource")
    @ConfigurationProperties("spring.datasource.write")
    public DataSource writeDataSource(){
        return DataSourceBuilder.create().build();
    }

    /**
     * 创建用于写入数据的JdbcTemplate对象。
     *
     * @param dataSource 写入数据源对象
     * @return 写入JdbcTemplate对象
     */
    @Bean(name = "writeJdbcTemplate")
    public JdbcTemplate writeJdbcTemplate(@Qualifier("writeDataSource")DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

}

```

**由于我们配置多数据源,因此在启动类时,需要移除默认数据源,让springboot加载我们的配置类`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`**

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@MapperScan(basePackages = "com.sangto.board.dao")
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication .class, args);
    }
}
```

service接口

```java
@Service
public class TaskServiceImpl implements TaskService {
    @Autowired
    @Qualifier("readJdbcTemplate")
    private JdbcTemplate readJdbcTemplate;

    @Autowired
    @Qualifier("writeJdbcTemplate")
    private JdbcTemplate writeJdbcTemplate;

    /**
     * 读出然后写入
     */
    @Override
    public void taskReadToWrite() { 
        List<ReadUser> readUsers = selectUsers();
        readUsers.forEach(e -> insertUser(e));
    } 

    List<ReadUser> selectUsers() {
        String sql = "select * from user";
        return readJdbcTemplate.query(sql,new ReadUserRowMapper());
    }

    void insertUser(ReadUser user) {
        String sql = "INSERT INTO user(id, name, age) VALUES(?, ?, ?)";
        writeJdbcTemplate.update(sql,
            user.getId(),
            user.getName(),
            user.getAge());
    }
}
```

## JdbcTemplate Plus: 动态加载数据源

定义`JdbcConfig.class` 工具类

```java
/**
 * @author: pangdi
 * @Date: 2023/5/18 14:13
 * @Description: jdbc连接工具类.
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class JdbcConfig {
    /**
     * 数据库链接地址
     */
    private String url;

    /**
     * 账户名
     */
    private String username;

    /**
     * 密码
     */
    private String password;

    /**
     * 驱动类型
     */
    private String driverClassName;

    /**
     * 数据库类型
     */
    private String dataBaseType;

    /**
     * 数据库名
     */
    private String databaseName;

    /**
     * url追加参数
     */
    private String parmString;
}

```

定义解密工具类`DesUtil.class`

```java
/**
 * @author: pangdi
 * @Date: 2023/5/23 15:44
 * @Description: 加密工具类
 */
@Component
public class DesUtil {
    /**
     * 秘钥
     */
    private static final String CUSTOMER_KEY = "Sangto01";
    private static final String IV = "union968";

    /**
     * 加密算法.
     *
     * @param src 数据源
     * @param key 秘钥,长度必须是8的倍数
     * @return 加密后的数据
     * @throws Exception 加密失败,向上抛出异常
     */
    public static String nonNullEncryptDes(final String src, final String key) throws Exception {
        if (!StrUtil.isEmpty(src)) {
            return encryptDes(src, key);
        }
        return null;
    }

    /**
     * 加密算法.
     *
     * @param src 数据源
     * @param key 秘钥,长度必须是8的倍数
     * @return 加密后的数据
     * @throws Exception 加密失败,向上抛出异常
     */
    public static String encryptDes(final String src, final String key) throws Exception {
        // 生成key,同时制定是des还是DESede,两者的key长度要求不同.
        final DESKeySpec desKeySpec = new DESKeySpec(key.getBytes(StandardCharsets.UTF_8));
        final SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        final SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
        // 加密向量
        final IvParameterSpec iv = new IvParameterSpec(IV.getBytes(StandardCharsets.UTF_8));
        final Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, iv);
        // 通过base64,将加密数组转换成字符串
        final byte[] b = cipher.doFinal(src.getBytes(StandardCharsets.UTF_8));
        return Base64.getEncoder().encodeToString(b);
    }

    /**
     * des 解密.
     *
     * @param src 数据源
     * @param key 秘钥
     * @return 解密数据
     * @throws Exception 解密失败,向上抛出异常
     */
    public static String decryptDes(final String src, final String key) throws Exception {
        // 通过base64,将字符串转换成byte数组
        final byte[] bytes = Base64.getDecoder().decode(src);
        // 解密key
        final DESKeySpec desKeySpec = new DESKeySpec(key.getBytes(StandardCharsets.UTF_8));
        final SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        final SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
        // 向量
        final IvParameterSpec iv = new IvParameterSpec(IV.getBytes(StandardCharsets.UTF_8));
        final Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, secretKey, iv);
        final byte[] retByte = cipher.doFinal(bytes);
        return new String(retByte);
    }


    /**
     * 自定义加密
     */
    public static String encryptDesByCustomer(final String src) throws Exception {
        return encryptDes(src, CUSTOMER_KEY);
    }

    /**
     * 自定义解密
     */
    public static String decryptDesByCustomer(final String src) throws Exception {
        return decryptDes(src, CUSTOMER_KEY);
    }

    // demo
    public static void main(String[] args) throws Exception {
        // 加密账号
        String encryptName = encryptDesByCustomer("root");
        System.out.println(encryptName);
        // 加密密码
        String encryptPsw = encryptDesByCustomer("root");
        System.out.println(encryptPsw);
    }
}
```

定义工具类`DataSourceUtil.class`

```java
/**
 * @author: pangdi
 * @Date: 2023/5/18 16:58
 * @Description: DataSource工具类
 */
public class DataSourceUtil {

    /**
     * 构造JdbcTemplate 和 PlatformTransactionManager.
     *
     * @return map集合
     */
    public static Map<String, Object> selectJdbcConfigMap(JdbcConfig jdbcConfig) throws Exception {
        Map<String, Object> map = new HashMap<>(3);
        // 构造数据源
        DataSource dataSource = constructDataSource(jdbcConfig);
        map.put("DataSource");
        // 创建 JdbcTemplate
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        map.put("JdbcTemplate");
        // 事务管理器
        PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        map.put("PlatformTransactionManager");
        return map;
    }

    /**
     * 构建数据请求url.
     *
     * @param jdbcConfig jdbc配置
     * @return 请求url
     */
    private static DriverManagerDataSource constructDataSource(JdbcConfig jdbcConfig) throws Exception {
        // 手动拼接url
        switch (jdbcConfig.getDataBaseType) {
            case "Mysql":
                // Mysql
                return constructMysqlUrl(jdbcConfig);
            case "Oracle":
                // Oracle
                break;
            case "SqlServer":
                // SqlServer
                break;
            default:
                throw new CustomerException(7104,"未知的数据库类型");
        }
        return null;
    }

    /**
     * 构建mysql数据源.
     *
     * @param jdbcConfig jdbc配置
     * @return mysql数据源
     */
    private static DriverManagerDataSource constructMysqlUrl(JdbcConfig jdbcConfig) throws Exception {
        // 用户名
        String username = DesUtil.decryptDesByCustomer(jdbcConfig.getUsername());
        // 密码
        String password = DesUtil.decryptDesByCustomer(jdbcConfig.getPassword());
        // url
        String url;
        if (StrUtil.isNotEmpty(jdbcConfig.getParmString())) {
            url = "jdbc:mysql://" +  jdbcConfig.getUrl() + "/" + jdbcConfig.getDatabaseName() + "?" + jdbcConfig.getParmString();
        } else {
            url = "jdbc:mysql://" +  jdbcConfig.getUrl() + "/" + jdbcConfig.getDatabaseName();
        }
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

}


```

使用

```java
        // jdbcUrl: jdbc:mysql://121.36.3.178:3306/test?useUnicode=true&characterEncoding=UTF-8&allowMuliQueries=true
        // username: root
        // password: root
        JdbcConfig jdbcConfig = new JdbcConfig()
                .setUrl("127.0.0.1:3306")
                .setUsername("root")
                .setPassword("root")
                .setDataBaseType("Mysql")
                .setDatabaseName("test")
                .setParmString("useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true");
        // JdbcMap
        Map<String, Object> jdbcMap = DataSourceUtil.selectJdbcConfigMap(targetJdbcConfig);
        // JdbcTemplate
        JdbcTemplate jdbcTemplate = (JdbcTemplate) jdbcMap .get("JdbcTemplate");
        // 事务管理器
        PlatformTransactionManager transactionManager = (PlatformTransactionManager) jdbcMap .get("PlatformTransactionManager");
        // DataSource
        DataSource dataSource = (DataSource) jdbcMap .get("DataSource");
```

service接口

```java
@Service
public class TaskServiceImpl implements TaskService {

    private JdbcTemplate readJdbcTemplate;

    private JdbcTemplate writeJdbcTemplate;

    initReadJdbcTemplate() {
        JdbcConfig jdbcConfig = new JdbcConfig()
                .setUrl("127.0.0.1:3306")
                .setUsername("root")
                .setPassword("root")
                .setDataBaseType("Mysql")
                .setDatabaseName("test")
                .setParmString("useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true");
        // JdbcMap
        Map<String, Object> jdbcMap = DataSourceUtil.selectJdbcConfigMap(targetJdbcConfig);
        // JdbcTemplate
        readJdbcTemplate = (JdbcTemplate) jdbcMap .get("JdbcTemplate");
    }

    initWriteJdbcTemplate() {
        JdbcConfig jdbcConfig = new JdbcConfig()
                .setUrl("127.0.0.1:3306")
                .setUsername("root")
                .setPassword("root")
                .setDataBaseType("Mysql")
                .setDatabaseName("test")
                .setParmString("useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true");
        // JdbcMap
        Map<String, Object> jdbcMap = DataSourceUtil.selectJdbcConfigMap(targetJdbcConfig);
        // JdbcTemplate
        writeJdbcTemplate= (JdbcTemplate) jdbcMap .get("JdbcTemplate");
    }

    /**
     * 读出然后写入
     */
    @Override
    public void taskReadToWrite() {
        initReadJdbcTemplate();
        initWriteJdbcTemplate();
        List<ReadUser> readUsers = selectUsers();
        readUsers.forEach(e -> insertUser(e));
    } 

    List<ReadUser> selectUsers() {
        String sql = "select * from user";
        return readJdbcTemplate.query(sql,new ReadUserRowMapper());
    }

    void insertUser(ReadUser user) {
        String sql = "INSERT INTO user(id, name, age) VALUES(?, ?, ?)";
        writeJdbcTemplate.update(sql,
            user.getId(),
            user.getName(),
            user.getAge());
    }
}
```

> 代码存放 git@gitee.com:guazixiong/practice_framework_code.git
>
> + springboot-mybatis-many-datasource  mybatis的多数据源配置
>
> + springboot-jdbctemplate-schedule   jdbctemplate的多数据源配置

