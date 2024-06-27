# Spring 中 @NotEmpty、@NotBlank、@NotNull，傻傻分不清楚！

常用的校验注解：

| 注解                           | 描述                                |
| ---------------------------- | --------------------------------- |
| `@NotNull`                   | 限制必须不为 null                       |
| `@NotEmpty`                  | 限制字符串、集合或数组必须不为 null 且不为空         |
| `@NotBlank`                  | 限制字符串必须不为空（不为 null、去除首位空格后长度不为 0） |
| `@Size(min, max)`            | 限制字符串、集合或数组的大小必须在指定的范围内           |
| `@Min(value)`                | 限制数字必须大于等于指定值                     |
| `@Max(value)`                | 限制数字必须小于等于指定值                     |
| `@DecimalMin(value)`         | 限制数字必须大于等于指定值                     |
| `@DecimalMax(value)`         | 限制数字必须小于等于指定值                     |
| `@Digits(integer, fraction)` | 限制数字必须符合指定的整数和小数位数                |
| `@Email`                     | 限制字符串必须是有效的 Email 地址              |
| `@Pattern(regex)`            | 限制字符串必须符合指定的正则表达式                 |
| `@Future`                    | 限制日期必须是未来的日期                      |
| `@Past`                      | 限制日期必须是过去的日期                      |
| `@AssertTrue`                | 限制布尔值必须为 true                     |
| `@AssertFalse`               | 限制布尔值必须为 false                    |

## 1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.0.5.RELEASE</version>
</dependency>
```

@NotEmpty、@NotBlank、@NotNull 包的位置：

`import javax.validation.constraints.*;`

## 2. 区别

##### @NotNull

适用于基本数据类型(Integer，Long，Double等等)，当 @NotNull 注解被使用在 String 类型的数据上，则表示**该数据不能为 Null（但是可以为 Empty**）

> 注：被其标注的字段可以使用 @size、@Max、@Min 对字段数值进行大小的控制

##### @NotBlank

适用于 String 类型的数据上，加了@NotBlank 注解的参数**不能为 Null 且 trim() 之后 size > 0，必须有实际字符**

##### @NotEmpty

适用于 String、Collection集合、Map、数组等等，加了@NotEmpty 注解的参数不能为 Null 或者 长度为 0

## 3. 使用方法

```java
@Data
public class BigPeople {
    @ApiModelProperty(value = "id" ,required = true)
    @NotNull(message = "id不能为空")
    @Length(message = "id不能超过{max}个长度",max = 10)
    private Integer id;

    @ApiModelProperty(value = "name" ,required = true)
    @NotBlank(message = "name不能为空")
    @Size(message = "名字最长为{max} 个字",max = 10)
    private String name;

    @ApiModelProperty(value = "age" ,required = true)
    @NotNull(message = "id不能为空")
    @Range(message = "age的长度范围为{min}岁到{max}岁之间",min = 5,max = 10)
    private Integer age;

    @ApiModelProperty(value = "treeNode" ,required = true)
    @NotEmpty(message = "treeNode不能为空")
    private List<String> treeNode;

}
```

@Valid 包位置：

```java
import javax.validation.Valid;
```

@Validated 包的位置

```java
import org.springframework.validation.annotation.Validated;

@ApiOperation(value = "新增或者修改一个人的信息")
@PostMapping("/updateOrInsert")
public Result updateOrInsert(@Valid @RequestBody  Person person){
    Boolean updateOrInsert = personService.updateOrInsert(person);
    if (updateOrInsert) {
        return new Result(ResultCode.SUCCESS,updateOrInsert);
    }
   return new Result(ResultCode.ERROR, "新增或者修改一个人的信息失败");
}

@ApiOperation(value = "新增或者修改一个人的信息")
@PostMapping("/updateOrInsert")
public Result updateOrInsert(@Validated @RequestBody  Person person){
    Boolean updateOrInsert = personService.updateOrInsert(person);
    if (updateOrInsert) {
        return new Result(ResultCode.SUCCESS,updateOrInsert);
    }
   return new Result(ResultCode.ERROR, "新增或者修改一个人的信息失败");
}
```

最上面三个注释：**必须需要搭配@Valid 或者@Validated使用，在检验Controller的入参**是否符合规范时

## 4. @Valid 和 @Validated 比较

最后我们来对 @Valid 和 @Validated 两个注解进行总结下：

1：**@Valid 和 @Validated 两者都可以对数据进行校验**，待校验字段上打的规则注解（@NotNull, @NotEmpty等）都可以对 @Valid 和 @Validated 生效；

2：**@Valid 进行校验的时候，需要用 BindingResult 来做一个校验结果接收**。**当校验不通过的时候，如果手动不 return ，则并不会阻止程序的执行；**

3：**@Validated 进行校验的时候，当校验不通过的时候，程序会抛出400异常**，阻止方法中的代码执行，这时**需要再写一个全局校验异常捕获处理类，然后返回校验提示**。

总体来说，**@Validated 使用起来要比 @Valid 方便一些**，它可以帮我们节省一定的代码，并且使得方法看上去更加的简洁。
