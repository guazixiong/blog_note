## Java注解

Java 定义的注解分三类。
（1）普通注解。
（2）元注解。
（3）自定义注解。
### 1、普通注解
普通注解在Java.lang 中有3个：

- `@Override`：检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- `@Deprecated`：标记过时方法。若某类或某方法加上该注解之后，表示此方法或类不再建议使用，调用时也会出现删除线，但并不代表不能用，只是说，不推荐使用，因为还有更好的方法可以调用。
- `@SuppressWarnings`：指示编译器去忽略注解中声明的警告。
### 2、元注解
元注解就是用来修饰注解的注解。通俗的理解为为了开发人员方便开发自定义注解的工具型注解，简单说的就类似JDK工具包作用。
在java.lang.annotation中有4个元注解：

```java
@Retention
```

用来说明注解的存活时间，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。保存方式有3种： 

   - RetentionPolicy.SOURCE - 标记的注释仅保留在源级别中，并由编译器忽略
   - RetentionPolicy.CLASS - 标记的注释在编译时由编译器保留，但Java虚拟机（JVM）会忽略
   - RetentionPolicy.RUNTIME - 标记的注释由JVM保留，因此运行时环境可以使用它
-  `@Documented`：标记这些注解是否包含在用户文档中，注释表明，无论何时使用指定的注释，都应使用Javadoc工具记录这些元素。（默认情况下，注释不包含在Javadoc中） 
```java
@Target
```

：标记这个注解应该是哪种 Java 对象范围，所修饰的对象范围有：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数），支持的元素类型有： 

   - ElementType.TYPE 可以应用于类的任何元素
   - ElementType.FIELD 可以应用于字段或属性
   - ElementType.METHOD 可以应用于方法级注释
   - ElementType.PARAMETER 可以应用于方法的参数
   - ElementType.CONSTRUCTOR 可以应用于构造函数
   - ElementType.LOCAL_VARIABLE 可以应用于局部变量
   - ElementType.ANNOTATION_TYPE 可以应用于注释类型
   - ElementType.PACKAGE 可以应用于包声明
   - ElementType.TYPE_PARAMETER 可以应用于类型参数
   - lementType.TYPE_USER 可以应用任何类型名称
-  `@Inherited`：此注释仅适用于类声明。 类继承关系中，子类会继承父类使用的注解中被`@Inherited`修饰的注解；接口继承关系中，子接口不会继承父接口中的任何注解，不管父接口中使用的注解有没有被`@Inherited`修饰；类实现接口关系中，类实现接口时不会继承任何接口中定义的注解。 

```java
     @Inherited
     @Target(ElementType.TYPE)
     @Retention(RetentionPolicy.RUNTIME)
     public @interface Person {
     }
 
     @Person
     public class Student {
     }
 
     public class ZhangSan extends Student{
     }
```

注解`@Person`被`@Inherited`修饰，`Student`被 `@Person`修饰，ZhangSan继承Student（ZhangSan上又无其他注解），那么ZhangSan就会拥有Person这个注解。
### 3、新增注解
Java7之后增加的3个注解：

- `@SafeVarargs`： Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- `@FunctionalInterface`：Java 8 开始支持，标识一个匿名函数或函数式接口。
- `@Repeatable`：Java 8 开始支持，标识某注解可以在同一个声明上使用多次。
### 3、自定义注解
开发者自己开发的注解。
## 实现自定义注解
### 环境准备
创建springboot工程，因为要测试Restful接口，需要引入`spring-boot-starter-web`
准备controller用来测试:
```java
 @RestController
 @RequestMapping("/api")
 public class ApiController {
 
     @RequestMapping("/checkTest01")
     public String checkTest01() {
         return "checkTest01";
     }
 
     @RequestMapping("/checkTest02")
     public String checkTest02() {
         return "checkTest02";
     }
 
     @RequestMapping("/checkTest03")
     public String checkTest03() {
         return "checkTest03";
     }
 }
```
### 自定义注解

```java
 package com.pbad.springboot.annotation;
 
 import java.lang.annotation.*;
 
 
 @Documented
 @Retention(RetentionPolicy.RUNTIME)
 @Target({ElementType.METHOD})
 public @interface ApiChecked {
 
     /**
      * 用来区分api的名字
      * @author pbad
      * @date 下午11:35 2022/2/5
      * @return java.lang.String
      **/
     String name() default "default";
 
     /**
      * 用来表示当前api开启限制标记
      * @author pbad
      * @date 下午11:35 2022/2/5
      * @return boolean
      **/
     boolean limit() default true;
 
     /**
      * 用来表示当前api等待时间
      * @author pbad
      * @date 下午11:36 2022/2/5
      * @return long
      **/
     long waitingTime() default 10;
 }
```
### 改造Controller接口

```java
 @RestController
 @RequestMapping("/api")
 public class ApiController {
 
     /**
      * 测试接口１，设置limit = false,其余为默认
      * @author pbad
      * @date 下午10:45 2022/2/6
      * @return java.lang.String
      **/
     @RequestMapping("/checkTest01")
     @ApiChecked(limit = false)
     public String checkTest01() {
         return "checkTest01";
     }
 
     /**
      * 测试接口2，设置name
      * @author pbad
      * @date 下午10:46 2022/2/6
      * @return java.lang.String
      **/
     @RequestMapping("/checkTest02")
     @ApiChecked(name = "check02",limit = true)
     public String checkTest02() {
         return "checkTest02";
     }
 
     /**
      * 测试接口3，设置waitingTime
      * @author pbad
      * @date 下午10:46 2022/2/6
      * @return java.lang.String
      **/
     @RequestMapping("/checkTest03")
     @ApiChecked(name = "check03",limit = true,waitingTime = 60)
     public String checkTest03() {
         return "checkTest03";
     }
 }
```
### 拦截器解析注解
#### 自定义拦截器

```java
 package com.pbad.springboot.intercepter;
 
 import com.pbad.springboot.annotation.ApiChecked;
 import lombok.extern.slf4j.Slf4j;
 import org.springframework.stereotype.Component;
 import org.springframework.web.method.HandlerMethod;
 import org.springframework.web.servlet.HandlerInterceptor;
 import javax.servlet.http.HttpServletRequest;
 import javax.servlet.http.HttpServletResponse;
 
 /**
  * 拦截器解析注解
  * @author pbad
  * @date 2022年02月05日 下午11:38
  */
 @Slf4j
 @Component
 public class ApiCheckedIntercepter implements HandlerInterceptor{
 
     /**
      * 前置拦截器
      * @author pbad
      * @date 下午11:43 2022/2/5
      * @param request
      * @param response
      * @param handler
      * @return boolean
      **/
     @Override
     public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
         log.info(" into intercepter");
         //注解相关数据验证
         if (!apiChecked(response,handler)){
             return false;
         }
         return true;
     }
 
     /**
      * 拦截器方式使用注解，解析注解参数
      * @author pbad
      * @date 下午11:47 2022/2/5 
      * @param response
      * @param handler 
      * @return boolean
      **/
     private boolean apiChecked(HttpServletResponse response, Object handler) throws Exception{
         // 反射获取方法上的LoginRequred注解
         HandlerMethod handlerMethod = (HandlerMethod)handler;
         ApiChecked interfaceChecked = handlerMethod.getMethod().getAnnotation(ApiChecked.class);
         
         // 注解为空，返回为true
         if(interfaceChecked == null){
             log.info("preHandle interfaceChecked is null ");
             return true;
         }
         
         // 注解是否开启限制(默认开启)
         if (interfaceChecked.limit()){
             //执行业务逻辑．．．．
             //这里直接输出注解参数，不实现具体的业务逻辑
             System.out.println(interfaceChecked.name());
             System.out.println(interfaceChecked.limit());
             System.out.println(interfaceChecked.waitingTime());
         }
         return true;
     }
 }
```
将自定义拦截器加入SpringBoot配置中去
```java
 /**
  * @author pbad
  * @date 2022年02月05日 下午11:54
  */
 @Configuration
 public class InterceptorTrainConfig implements WebMvcConfigurer {
 
     /**
      * 拦截器注册类 InterceptorRegistry 加入自己的拦截器
      * @param registry
      */
     @Override
     public void addInterceptors(InterceptorRegistry registry) {
         // 加入自己的拦截器,针对/api下的所有请求
         registry.addInterceptor(new ApiCheckedIntercepter()).addPathPatterns("/api/**");
     }
 }
```
#### 注解测试
启动项目，访问`http://localhost:8080/api/checkTest01`,测试接口１
```bash
 checkTest01
 
 Console:
```
访问`http://localhost:8080/api/checkTest02`，测试接口２
```bash
 checkTest01
 
 Console:
 check02
 true
 10
```
访问`http://localhost:8080/api/checkTest03`，测试接口３
### Aop实现
> 使用Aop实现时，请先将拦截器配置移除，避免影响测试结果

#### 引入AOP
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```
#### AOP切面类
```java
package com.pbad.springboot.aspect;

import com.pbad.springboot.annotation.ApiChecked;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * 切面类
 * @author pbad
 * @date 2022年02月06日 下午10:20
 */
@Slf4j
@Aspect
@Component
public class LimitCheckedAspect {

    @Order(1) // Order 代表优先级，数字越小优先级越高
    //定义切入点
    @Pointcut("@annotation(com.pbad.springboot.annotation.ApiChecked)")
    public void checkedPoint(){};

    /**
     * 环绕通知,针对注解ApiChecked进行处理
     * @author pbad
     * @date 下午10:57 2022/2/6
     * @param joinPoint
     * @param apiChecked
     * @return java.lang.Object
     **/
    @Around(value = "checkedPoint() && @annotation(apiChecked)")
    public Object doLogs(ProceedingJoinPoint joinPoint, ApiChecked apiChecked) throws Throwable {
        // 获取方法名称
        String methodName = joinPoint.getSignature().getName();
        // 获取入参
        Object[] param = joinPoint.getArgs();
        System.out.println("方法名称："+methodName);
        System.out.println("参数");
        for (Object o : param) {
            System.out.println(o);
        }
        log.debug("methodName {}, param {}", methodName, param);

        // 注解拦截处理
        System.out.println(apiChecked.name());
        System.out.println(apiChecked.limit());
        System.out.println(apiChecked.waitingTime());

        //执行原有接口方法
        Object result = null;
        result = joinPoint.proceed();
        return result;
    }
}
```
#### 注解测试
启动项目，访问`http://localhost:8080/api/checkTest01`，测试接口１
```bash
checkTest01

Console:
方法名称：checkTest01
参数
default
false
10
```
访问`http://localhost:8080/api/checkTest02`，测试接口2
```bash
checkTest02

Console:
方法名称：checkTest02
参数
check02
true
10
```
访问`http://localhost:8080/api/checkTest03`，测试接口３
```bash
checkTest02

Console控制台:
方法名称：checkTest03
参数
check03
true
60
```
## 参考链接

- (ฅ>ω<*ฅ) 噫又好啦 ~基于SpringBoot的自定义注解 | 张小菜苔https://small-rose.github.io/posts/ab810780.html#toc-heading-1
