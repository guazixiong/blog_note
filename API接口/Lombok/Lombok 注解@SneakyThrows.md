# @SneakyThrows - Project Lombok

>  为了大胆地在以前没有抛出过的地方抛出受检异常！

**概述 `@SneakyThrows` 可以用于在不在方法的 throws 子句中声明的情况下，偷偷地抛出受**

**检异常**。当然，这种具有争议的能力应谨慎使用。

Lombok 生成的代码不会忽略、包装、替换或以其他方式修改被抛出的受检异常；它只是欺

骗了编译器。**在 JVM（类文件）级别上，所有异常，无论是否受检，都可以被抛出，不受方**

**法的 throws 子句的限制**，这就是为什么它能够工作的原因。

使用 @SneakyThrows 的常见用例主要涉及两种情况下，您希望退出受检异常机制：

+ 一个不必要严格的接口，比如 Runnable - 无论您的 run() 方法传播出什么异常，无论是
  
  受检异常还是非受检异常，它都将传递给线程的未处理异常处理器。捕获受检异常并
  
  其包装在某种 RuntimeException 中只会掩盖真正的问题原因。

+ 一种 "不可能" 的异常。例如，`new String(someByteArray, "UTF-8")` 声明它可能抛
  
  出 UnsupportedEncodingException，但根据 JVM 规范，UTF-8 必须始终可用。在这
  
  里，出现 UnsupportedEncodingException 的可能性与使用 String 对象时出现 
  
  ClassNotFoundError 一样小，而您也不会捕获这些异常！

在使用 lambda 语法（arg -> action）时，受不必要严格接口的限制特别常见；

然而，lambda 无法被注解，这意味着很难与 @SneakyThrows 结合使用。

请注意，直接捕获被偷偷抛出的受检类型是不可能的，因为 **javac 不允许您为在 try 体中没**

**有声明为抛出的异常类型编写 catch 块**。这个问题在上述任何一个用例中都不相关，**因此这**

**应该作为一个警告，告诉您不要在没有仔细考虑的情况下使用 @SneakyThrows 机制！**

**您可以向 `@SneakyThrows` 注解传递任意数量的异常。如果不传递异常，您可以偷偷地抛出**

**任何异常。**

### Demo

```java
import lombok.SneakyThrows;

public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }

  @SneakyThrows
  public void run() {
    throw new Throwable();
  }
}
```

```java
import lombok.Lombok;

public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }

  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      throw Lombok.sneakyThrow(t);
    }
  }
}
```

## 支持的配置键：

lombok.sneakyThrows.flagUsage = [warning | error]（默认值：未设置）
如果进行配置，Lombok 将标记对 @SneakyThrows 的任何使用为警告或错误。

## 小细则

因为 @SneakyThrows 是实现细节而不是方法签名的一部分，所以如果您尝试声明一个受检

异常作为 sneakily 抛出的异常，而您又没有调用任何抛出该异常的方法，那么这是一个错误

（为了适应子类，throws 语句可以合法地这样做）。同样，@SneakyThrows 不具有继承

性。

对于持反对态度的人来说：默认情况下，Eclipse 将为未捕获的异常提供一个“快速修复”选

项，该选项将错误的语句包装在一个 try/catch 块中，catch 块中只有 e.printStackTrace()。

与偷偷地将异常向下抛出相比，这种方式非常低效，因此 Roel 和 Reinier 认为受检异常系统

远非完美，因此需要一种退出机制。

如果在构造函数上使用 @SneakyThrows，那么对兄弟或父类构造函数的调用将不会受到 

@SneakyThrows 的处理。这是我们无法解决的 Java 限制：对兄弟/父类构造函数的调用必

须是构造函数中的第一个语句；它们不能放在 try/catch 块内部。

**对于一个空方法或仅包含对兄弟/父类构造函数调用的构造函数，@SneakyThrows 不会生成 try/catch 块，但会产生警告。**

## 原文链接

+ [@SneakyThrows](https://projectlombok.org/features/SneakyThrows)
