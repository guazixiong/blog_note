# 记Spring yaml配置参数通过@Value获取异常问题

## 前提

在Spring项目的`application.yaml`文件中配置了一个值：

```yaml
some:
  key: 00510103
```

使用`@Value`注解引用该值时：

```java
@Value("${some.key}")
private String someKey;
```

但实际引用的值变成了`168003`。

## 问题原因

配置值的格式问题;

### YAML的数字解释规则

YAML在解释数字时，会依据其格式决定数字的进制：

- 以 `0` 开头的数字会被解释为八进制数。
- 以 `0x` 开头的数字会被解释为十六进制数。
- 其他情况下，数字会被解释为十进制数。

因此，当在YAML文件中写入 `00510103` 时，YAML解释器会将其解释为八进制数 `00510103`，然后将其转换为十进制数 `168003`。

###### 执行步骤，除以 16 得到余数,= (510103)8

| 除以 8     | 商    | 余数 | 位次 |
| ---------- | ----- | ---- | ---- |
| (168003)/8 | 21000 | 3    | 0    |
| (21000)/8  | 2625  | 0    | 1    |
| (2625)/8   | 328   | 1    | 2    |
| (328)/8    | 41    | 0    | 3    |
| (41)/8     | 5     | 1    | 4    |
| (5)/8      | 0     | 5    | 5    |

在YAML中，数字如果以零开头，会被解释为八进制数。这个解释规则是YAML语言本身的一部分，而不是Spring特有的行为。

### 其他示例

```yaml
number1: 00510103
number2: 168003
number3: 0x1A
```

+ `number1` 被解释为八进制数 `00510103`，其十进制值为 `168003`。

+ `number2` 被解释为十进制数 `168003`。

+ `number3` 被解释为十六进制数 `0x1A`，其十进制值为 `26`。

### Spring读取配置值

当Spring通过`@Value`注解从YAML文件中读取配置值时，读取的就是经过YAML解释器解释后的值。因此，如果YAML解释器将某个值解释为八进制数，Spring读取的就是该八进制数对应的十进制值。

## 解决方案

明确指定值的类型为字符串; 

为了避免YAML解释器将`00510103`解释为八进制数，可以明确指定值的类型为字符串。使用引号将值包裹起来：

```yaml
some:
  key: "00510103"
```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class SomeComponent {

    @Value("${some.key}")
    private String someKey;

    public void printSomeKey() {
        System.out.println("The value of some.key is: " + someKey);
    }
}
```

