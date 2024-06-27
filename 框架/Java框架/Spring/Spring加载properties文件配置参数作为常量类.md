# Spring加载properties文件配置参数作为常量类

properties参数

```properties
user.name = zhangsna
user.age = 15
user.sex = 1
```

常量类UserConstant

```java
@Component
public class UserConstant{
    /**
     * 姓名
     */
    private final String name;
    /**
     * 年龄
     */
    private final int age;
    /**
     * 性别
     */
    private final String sex;

    public PurchaseShopnoConstant(
            @Value("${user.name}") String name,
            @Value("${user.age}") int age,
            @Value("${user.sex}") String sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    /**
     * 姓名是否匹配.
     *
     * @param name 姓名
     * @return true:匹配,false:不匹配
     */
    public boolean isMatchName(String name) {
        if (name == null || "".equals(name)) {
            throw new RuntimeException("姓名不能为空");
        }
        return this.name.equals(name);
    }
}
```

使用

```java
String name = UserConstant.name;
int age = UserConstant.age;
String sex = UserConstant.sex;
boolean flag = UserConstant.isMatchName("lisi");
```