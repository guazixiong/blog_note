_**Mockito 是一个模拟测试框架，主要功能是在单元测试中模拟类/对象的行为。**_

# 为什么要使用Mockito?
Mock可以理解为创建一个虚假的对象,或者说模拟出一个对象.在测试环境中用来替换掉真实的对象,以达到我们可以

1. 验证该对象的某些方法的调用情况,调用了多少次,参数是多少.
2. 给这个对象的行为做一个定义,来指定返回结果或指定特定的动作.
# Mockito数据隔离
根据 JUnit 单测隔离 ，当 Mockito 和 JUnit 配合使用时，也会将非static变量或者非单例隔离开。
比如使用 @Mock 修饰的 mock 对象在不同的单测中会被隔离开。
> 比如使用 @Mock 修饰的 mock 对象在不同的单测中会被隔离开。

# Mock方法
mock方法来自org.mockito.Mock,它标识可以mock一个对象或者是接口
```java
public static <T> mock(Class<T> classToMock);
```

- classToMock:待mock对象的class类
- 返回mock出来的类
```java
Random random = Mockito.mock(Random.class);
```
# Stub存根
存根的意思就是给mock对象规定一行的行为,使其按照我们的要求来执行具体的动作;
```java

 //You can mock concrete classes, not just interfaces
 LinkedList mockedList = mock(LinkedList.class);

 //stubbing
 when(mockedList.get(0)).thenReturn("first");
 when(mockedList.get(0)).thenReturn("two");
 when(mockedList.get(1)).thenThrow(new RuntimeException());

 //following prints "two"
 System.out.println(mockedList.get(0));

 //following throws runtime exception
 System.out.println(mockedList.get(1));

 //following prints "null" because get(999) was not stubbed
 System.out.println(mockedList.get(999));

 //Although it is possible to verify a stubbed invocation, usually it's just redundant
 //If your code cares what get(0) returns, then something else breaks (often even before verify() gets executed).
 //If your code doesn't care what get(0) returns, then it should not be stubbed.
 verify(mockedList).get(0);
```
## 常用方法

- when().Return() 与 doReturn() 设置方法的返回值
- when().thenthrow() 与 doThrow() 让方法抛出异常
- doNothing() 让void函数什么都不做
- doAnswer()自定义方法处理逻辑
- thenCallRealMethod()调用 spy 对象的真实方法
### when().Return() 与 doReturn() 设置方法的返回值
```java
Mockito.when(mock.someMethod("some args")).Return("result");

Mockito.doReturn("result").when(mock).someMethod("some arg");
```
### when().thenthrow() 与 doThrow() 让方法抛出异常
```java
// 只针对返回值非void的函数
Mockito.when(mock.someMethod("some args")).thenthrow(new Exception("自定义异常"));
// 通用
Mockito.doThrow(new Exception("自定义异常"))
    .when(mock)
    .someMethod("some arg");
```
### doNothing() 让void函数什么都不做
```java
Mockito.doNothing().when(mock).someMethod("some args");
```
### doAnswer()自定义方法处理逻辑
```java
     // 自定义返回值thenAnswer()
     when(mock.someMethod(anyString())).thenAnswer(
         new Answer() {
             public Object answer(InvocationOnMock invocation) {
                 Object[] args = invocation.getArguments();
                 Object mock = invocation.getMock();
                 return "called with arguments: " + Arrays.toString(args);
             }
     });
    
     //Following prints "called with arguments: [foo]"
     System.out.println(mock.someMethod("foo"));
```
### thenCallRealMethod()调用 spy 对象的真实方法
```java
Mockito.when(mock.someMethod("some args")).thenCallRealMethod();

Mockito.doCallRealMethod().when(mock).someMethod("some arg");
```
## 链式存根
```java
// 最后一次的Stub将覆盖第一次的存根
 when(mock.someMethod("some arg"))
   .thenReturn("one")
 when(mock.someMethod("some arg"))
   .thenReturn("two")
when(mock.someMethod("some arg")).thenReturn("one", "two", "three");
     
//First call: prints "two"
System.out.println(mock.someMethod("some arg"));
//Second call: prints "two"
System.out.println(mock.someMethod("some arg"));
```
```java
 // 第一次存根时,抛出异常,第二次返回foo值
 when(mock.someMethod("some arg"))
   .thenThrow(new RuntimeException())
   .thenReturn("foo");

 //First call: throws runtime exception:
 mock.someMethod("some arg");

 //Second call: prints "foo"
 System.out.println(mock.someMethod("some arg"));

 //Any consecutive call: prints "foo" as well (last stubbing wins).
 System.out.println(mock.someMethod("some arg"));
```
## 使用then、thenAnswer 自定义方法处理逻辑
实现 Answer 接口的对象，在改对象中可以获取调用参数，自定义返回值
```java
     // 自定义返回值thenAnswer()
     when(mock.someMethod(anyString())).thenAnswer(
         new Answer() {
             public Object answer(InvocationOnMock invocation) {
                 Object[] args = invocation.getArguments();
                 Object mock = invocation.getMock();
                 return "called with arguments: " + Arrays.toString(args);
             }
     });
    
     //Following prints "called with arguments: [foo]"
     System.out.println(mock.someMethod("foo"));


    @Test
    void testDemo02() {
      Mockito.when(studentService.getStudentByUserName("张三")).thenAnswer(
          (Answer<Student>) invocationOnMock -> new Student("赵六","13215522144","河南省")
      );
      Student student = studentService.getStudentByUserName("张三");
      // prints: Student{username='赵六', phone='13215522144', address='河南省'}
      System.out.println(student.toString());
    }
```
## reset()方法，可以重置之前自定义的返回值和异常
```java
import org.junit.Assert;
import org.junit.Test;

import static org.mockito.Mockito.*;

public class MockitoDemo {

    static class ExampleService {

        public int add(int a, int b) {
            return a+b;
        }

    }

    @Test
    public void test() {

        ExampleService exampleService = mock(ExampleService.class);

        // mock 对象方法的默认返回值是返回类型的默认值
        Assert.assertEquals(0, exampleService.add(1, 2));

        // 设置让 add(1,2) 返回 100
        when(exampleService.add(1, 2)).thenReturn(100);
        Assert.assertEquals(100, exampleService.add(1, 2));

        // 重置 mock 对象，add(1,2) 返回 0
        reset(exampleService);
        Assert.assertEquals(0, exampleService.add(1, 2));

    }

}

```
```java
import org.junit.Assert;
import org.junit.Test;

import static org.mockito.Mockito.*;

public class MockitoDemo {

    static class ExampleService {

        public int add(int a, int b) {
            return a+b;
        }

    }

    @Test
    public void test() {

        ExampleService exampleService = spy(new ExampleService());

        // spy 对象方法调用会用真实方法，所以这里返回 3
        Assert.assertEquals(3, exampleService.add(1, 2));

        // 设置让 add(1,2) 返回 100
        when(exampleService.add(1, 2)).thenReturn(100);
        Assert.assertEquals(100, exampleService.add(1, 2));

        // 重置 spy 对象，add(1,2) 返回 3
        reset(exampleService);
        Assert.assertEquals(3, exampleService.add(1, 2));

    }

}

```
## 使用 mockingDetails 判断对象是否为 mock对象、spy 对象
Mockito 的 mockingDetails 方法会返回 MockingDetails 对象，它的 isMock 方法可以判断对象是否为 mock 对象，isSpy 方法可以判断对象是否为 spy 对象。
```java
import org.junit.Test;

import static org.mockito.Mockito.*;

public class MockitoDemo {

    static class ExampleService {

        public int add(int a, int b) {
            return a+b;
        }

    }

    @Test
    public void test() {

        ExampleService exampleService = mock(ExampleService.class);

        // 判断 exampleService 是否为 mock 对象
        System.out.println( mockingDetails(exampleService).isMock() );     // true

        // 判断 exampleService 是否为 spy 对象
        System.out.println( mockingDetails(exampleService).isSpy() );      // false

    }

}

```
## 注意事项
> - By default, for all methods that return a value, a mock will return either null, a primitive/primitive wrapper value, or an empty collection, as appropriate. For example 0 for an int/Integer and false for a boolean/Boolean. 
> - Stubbing can be overridden: for example common stubbing can go to fixture setup but the test methods can override it. Please note that overridding stubbing is a potential code smell that points out too much stubbing
> - Once stubbed, the method will always return a stubbed value, regardless of how many times it is called. 
> - Last stubbing is more important - when you stubbed the same method with the same arguments many times. Other words: **the order of stubbing matters** but it is only meaningful rarely, e.g. when stubbing exactly the same method calls or sometimes when argument matchers are used, etc.

- **默认情况下，对于所有返回值的方法，模拟将酌情返回 null、原语/原语包装器值或空集合。**例如，0表示 int/Integer，false 表示 Boolean/Boolean
- 存根可以被覆盖: 例如，常见的存根可以设置，但是测试方法可以覆盖它。请注意，重写存根是一种潜在的代码行为，指出太多的存根
-  一旦设置存根，无论调用多少次，该方法总是返回存根值
- 最后一次存根更重要——当你用同样的参数多次撞击同一个方法的时候:存根**的顺序很重要** 但它很少有意义，例如，当存根完全相同的方法调用或有时当使用参数匹配器时，等等
# 参数匹配
```java
import org.junit.Assert;
import org.junit.Test;

import java.util.List;

import static org.mockito.Mockito.*;

public class MockitoDemo {


    @Test
    public void test() {
        List mockList = mock(List.class);

        Assert.assertEquals(0, mockList.size());
        Assert.assertEquals(null, mockList.get(0));

        mockList.add("a");  // 调用 mock 对象的写方法，是没有效果的

        Assert.assertEquals(0, mockList.size());      // 没有指定 size() 方法返回值，这里结果是默认值
        Assert.assertEquals(null, mockList.get(0));   // 没有指定 get(0) 返回值，这里结果是默认值

        when(mockList.get(0)).thenReturn("a");          // 指定 get(0)时返回 a

        Assert.assertEquals(0, mockList.size());        // 没有指定 size() 方法返回值，这里结果是默认值
        Assert.assertEquals("a", mockList.get(0));      // 因为上面指定了 get(0) 返回 a，所以这里会返回 a

        Assert.assertEquals(null, mockList.get(1));     // 没有指定 get(1) 返回值，这里结果是默认值

        // 精确匹配
        when(mockStringList.get(0)).thenReturn("a");
        when(mockStringList.get(1)).thenReturn("b");

        Assert.assertEquals("a", mockStringList.get(0));
        Assert.assertEquals("b", mockStringList.get(1));

        // 模糊匹配
        when(mockStringList.get(anyInt())).thenReturn("a");  // 使用 Mockito.anyInt() 匹配所有的 int
        
        Assert.assertEquals("a", mockStringList.get(0)); 
        Assert.assertEquals("a", mockStringList.get(1));
    }
}

```
# @Mock 注解
`@mock`快速创建mock的方法,使用
`@mock`注解需要搭配`MockitoAnnotations.openMocks(testClass)`方法一起使用.
```java
   public class ArticleManagerTest {

       @Mock 
       private ArticleCalculator calculator;
       @Mock 
       private ArticleDatabase database;
       @Mock 
       private UserProvider userProvider;

       private ArticleManager manager;

    
       @org.junit.jupiter.api.Test
       void testSomethingInJunit5() {
           // 初始化mock对象
           MockitoAnnotations.openMocks(testClass);
           //Mockito.mock(class);
           Mockito.spy(class);
       }

       // 简化
       // @BeforeEach
       // void setUp() {
       //     MockitoAnnotations.openMocks(this);
       // }
   
   }
```
# Spy方法与@Spy注解 
spy 和 mock不同，不同点是：

- spy 的参数是对象实例，mock 的参数是 class。
- 被 spy 的对象，调用其方法时默认会走真实方法。mock 对象不会。

`@Spy`注解需要搭配`MockitoAnnotations.openMocks(testClass)`方法一起使用.
```java
import org.junit.Assert;
import org.junit.Test;
import static org.mockito.Mockito.*;


class ExampleService {

    int add(int a, int b) {
        return a+b;
    }

}

public class MockitoDemo {

    // 测试 spy
    @Test
    public void test_spy() {

        ExampleService spyExampleService = spy(new ExampleService());

        // 默认会走真实方法
        Assert.assertEquals(3, spyExampleService.add(1, 2));

        // 打桩后，不会走了
        when(spyExampleService.add(1, 2)).thenReturn(10);
        Assert.assertEquals(10, spyExampleService.add(1, 2));

        // 但是参数比匹配的调用，依然走真实方法
        Assert.assertEquals(3, spyExampleService.add(2, 1));

    }

    // 测试 mock
    @Test
    public void test_mock() {

        ExampleService mockExampleService = mock(ExampleService.class);

        // 默认返回结果是返回类型int的默认值
        Assert.assertEquals(0, mockExampleService.add(1, 2));

    }


}

```
spy 对应注解 @Spy，和 @Mock 是一样用的。
```java
import org.junit.Assert;
import org.junit.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;

import static org.mockito.Mockito.*;


class ExampleService {

    int add(int a, int b) {
        return a+b;
    }

}

public class MockitoDemo {

    @Spy
    private ExampleService spyExampleService;

    @Test
    public void test_spy() {

        MockitoAnnotations.openMocks(this);

        Assert.assertEquals(3, spyExampleService.add(1, 2));

        when(spyExampleService.add(1, 2)).thenReturn(10);
        Assert.assertEquals(10, spyExampleService.add(1, 2));

    }

}


```
# @InjectMocks 注解注入 mock 对象
mockito 会将 @Mock、@Spy 修饰的对象自动注入到 @InjectMocks 修饰的对象中。
注入方式有多种，mockito 会按照下面的顺序尝试注入：

1. 构造函数注入
2. 设值函数注入（set函数）
3. 属性注入
```java
package demo;

import java.util.Random;

public class HttpService {

    public int queryStatus() {
        // 发起网络请求，提取返回结果
        // 这里用随机数模拟结果
        return new Random().nextInt(2);
    }

}

```
```java
package demo;

public class ExampleService {

    private HttpService httpService;

    public String hello() {
        int status = httpService.queryStatus();
        if (status == 0) {
            return "你好";
        }
        else if (status == 1) {
            return "Hello";
        }
        else {
            return "未知状态";
        }
    }

}
```
```java
import org.junit.Assert;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.mockito.Mockito.when;


public class ExampleServiceTest {

    @InjectMocks // 将httpService主动注入
    private ExampleService exampleService = new ExampleService();

    @Mock
    private HttpService httpService;

    @Test
    public void test01() {

        MockitoAnnotations.initMocks(this);

        when(httpService.queryStatus()).thenReturn(0);

        Assert.assertEquals("你好", exampleService.hello());

    }

}

```
# 验证和断言
## verify
验证是校验待验证的对象是否发生过某些行为,Mockito中验证的方法是:`verify`
```java
verify(mock).someMoethod("some arg");
verify(mock,times(100)).someMoethod("some arg");
```
使用verify验证(Junit的断言机制):

```java
	@Test
    void check() {
        Random random = Mockito.mock(Random.class);
        System.out.println(random.nextInt());
        Mockito.verify(random,Mockito.times(2)).nextInt();
    }
```
Verify配合times()方法,可以校验某些操作发生的次数
```java
 //using mock
 mockedList.add("once");

 mockedList.add("twice");
 mockedList.add("twice");

 mockedList.add("three times");
 mockedList.add("three times");
 mockedList.add("three times");

 // 默认调用1次
 verify(mockedList).add("once");
 verify(mockedList, times(1)).add("once");

 // 自定义调用多次
 verify(mockedList, times(2)).add("twice");
 verify(mockedList, times(3)).add("three times");

 // 从不调用
 verify(mockedList, never()).add("never happened");

 // atLeast()/atMost()  至少调用 / 之多调用
 verify(mockedList, atMostOnce()).add("once");
 verify(mockedList, atLeastOnce()).add("three times");
 verify(mockedList, atLeast(2)).add("three times");
 verify(mockedList, atMost(5)).add("three times");

 // 超时验证
 verify(mock, timeout(100)).someMethod();
 verify(mock, timeout(100).times(1)).someMethod();

 //只要 someMethod() 在 100 毫秒内被调用 2 次，就会通过
 verify(mock, timeout(100).times(2)).someMethod();

 //someMethod() 至少在 100 毫秒内被调用 2 次，就会通过
 verify(mock, timeout(100).atLeast(2)).someMethod();

```
## 自定义验证失败消息
```java
 // 将在验证失败时打印自定义消息
 verify(mock, description("This will print on failure")).someMethod();

 // 适用于任何验证模式
 verify(mock, times(2).description("someMethod should be called twice")).someMethod();
```
### 校验mock对象是否发生交互
```java
 //使用mock对象 - 只有 mockOne 是可交互的
 mockOne.add("one");

 //普通验证
 verify(mockOne).add("one");

 //验证某个方法从未被调用
 verify(mockOne, never()).add("two");

 //验证其他方法从未发送过交互
 verifyZeroInteractions(mockTwo, mockThree);
```
### 寻找多余的调用
> 一句**警告**：一些做过很多经典的、`expect-run-verify mock`的用户往往会经常使用`verifyNoMoreInteractions()`，即使在每个测试方法中也是如此。 verifyNoMoreInteractions()不建议在每个测试方法中使用。 verifyNoMoreInteractions()是来自交互测试工具包的一个方便的断言。仅在相关时使用它。滥用它会导致**过度指定**、**不易维护的**测试。

```java
 //使用mock对象
 mockedList.add("one");
 mockedList.add("two");

 verify(mockedList).add("one");

 //下面的验证会失败
 verifyNoMoreInteractions(mockedList);
```
# 临时 mock 对象
如果需要**临时**将一个对象的内部对象替换为 mock 对象，在无法通过set和get处理内部对象的情况下，可以利用反射搞定。
[Java JOOR 反射库](https://www.letianbiji.com/java/java-joor.html) 是一个很好用的反射库。本文用它进行临时替换。
```java
.
├── build.gradle
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── demo
    │   │       ├── HttpService.java
    │   │       └── BizService.java
    │   └── resources
    └── test
        ├── java
        │   └── demo
        │       └── BizServiceTest.java
        └── resources

```
```java
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    testCompile group: 'org.mockito', name: 'mockito-core', version: '2.25.1'
    testCompile group: 'org.jooq', name: 'joor-java-8', version: '0.9.7'
}
```
```java
package demo;

public class HttpService {

    public int queryStatus() {
        // 发起网络请求，提取返回结果
        // 这里直接返回0
        return 0;
    }

}
```
```java
package demo;

public class BizService {

    private HttpService httpService = new HttpService();

    public String hello() {
        int status = httpService.queryStatus();
        if (status == 0) {
            return "你好";
        }
        else if (status == 1) {
            return "Hello";
        }
        else {
            return "未知状态";
        }
    }

}
```
```java
package demo;

import org.joor.Reflect;
import org.junit.Test;

import static org.mockito.Mockito.*;

public class BizServiceTest {

    private BizService bizService = new BizService();

    @Test
    public void testHello() {

        System.out.println( bizService.hello() );  // 输出'你好'

        // 取出原有的对象
        Object realHttpService = Reflect.on(bizService).get("httpService");

        // 创建 mock 对象，并用它替换掉 bizService 中的 httpService 对象
        HttpService mockHttpService = mock(HttpService.class);
        when(mockHttpService.queryStatus()).thenReturn(1);
        Reflect.on(bizService).set("httpService", mockHttpService);

        System.out.println( bizService.hello() );  // 输出'hello'

        // 再将原先的对象设置回去
        Reflect.on(bizService).set("httpService", realHttpService);
        System.out.println( bizService.hello() );  // 输出'你好'

    }

}
```
# Mockito Demo实例
[GitHub - guazixiong/spring-unit-testing-with-junit-and-mockito: Spring Unit Testing with JUnit and Mockito](https://github.com/guazixiong/spring-unit-testing-with-junit-and-mockito)

