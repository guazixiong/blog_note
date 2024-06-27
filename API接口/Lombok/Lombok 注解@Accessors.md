# lombok @Accessors用法详解

- @Accessors(chain=true)
- @Accessors(fluent = true)
- @Accessors(prefix = 'f')
- 坑

## Lombok @Accessors

### @Accessors(chain=true)

链式访问，该注解设置`chain=true`，生成setter方法返回this（***也就是返回的是对象\***），代替了默认的返回void。

```java
package com.pollyduan;

import lombok.Data;
import lombok.experimental.Accessors;

@Data
@Accessors(chain=true)
public class User {
    private Integer id;
    private String name;
    private Integer age;

    public static void main(String[] args) {
        //开起chain=true后可以使用链式的set
        User user=new User().setAge(31).setName("pollyduan");//返回对象
        System.out.println(user);
    }

}
```

### @Accessors(fluent = true)

与chain=true类似，区别在于getter和setter不带set和get前缀。

```java
package com.pollyduan;

import lombok.Data;
import lombok.experimental.Accessors;

@Data
@Accessors(fluent=true)
public class User {
    private Integer id;
    private String name;
    private Integer age;

    public static void main(String[] args) {
        //fluent=true开启后默认chain=true，故这里也可以使用链式set
        User user=new User().age(31).name("pollyduan");//不需要写set
        System.out.println(user);
    }

}
```

### @Accessors(prefix = "f")

set方法忽略指定的前缀。不推荐大神们这样去命名。

```java
package com.pollyduan;

import lombok.Data;
import lombok.experimental.Accessors;

@Data
@Accessors(prefix = "f")
public class User {
    private String fName = "Hello, World!";

    public static void main(String[] args) {
        User user=new User();
        user.setName("pollyduan");//注意方法名
        // user.setFName("pollyduan") 前缀被省略
        System.out.println(user);
    }

}
```

## 坑

### EasyExcel与@Accessors(chain = true)不兼容

> 使用`@Accessors(chain = true)`后,EasyExcel read获取数值为null

注解设置`chain=true`，生成setter方法返回this（***也就是返回的是对象\***），代替了默认的返回void。EasyExcel通过注解`@ExcelProperty()`获取`set()`方法,所以数据返回为null.

![img](Lombok%20%E6%B3%A8%E8%A7%A3@Accessors.assets/202305042330562-170031623181690.png)

## 参考

* [lombok @Accessors用法详解（一看就能懂） - 简书](https://www.jianshu.com/p/67a15b2e4a92)
* [EasyExcel与@Accessors(chain = true)不兼容分析_使用@accessor用easyescel失败_milu、的博客-CSDN博客](https://blog.csdn.net/qq_28036249/article/details/108035369)