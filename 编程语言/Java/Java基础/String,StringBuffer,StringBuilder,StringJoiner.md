# String,StringBuffer,StringBuilder,StringJoiner 还傻傻分不清楚

## String

在Java中，`String`是一个引用类型，它本身也是一个`class`。但是，Java编译器对`String`有特殊处理，即可以直接用`"..."`来表示一个字符串：

```java
String s1 = "Hello!";
```

字符串在`String`内部是通过一个`char[]`数组表示的

```java
String s2 = new String(new char[] {'H', 'e', 'l', 'l', 'o', '!'});
```

**String 是 不可变的(immutable)的，即除非利用反射强制修改它的值，否则一个 String 对象一旦被创建，其值就不会被修改。**

> String 的 replace、concat 和 substring 方法，都会返回一个新的 String 对象，而不是对原来的 String 对象进行修改。

Java 使 String 类为 immutable 的实现方式是：String 类是 final 的，不可以被继承，而且其成员属性（property）都是 private 的，也没有 public 的方法会对字符串的值（String 中的 value 属性进行修改）。

## StringBuffer

为了应对频繁对字符串做修改操作的场景，Java 从 JDK1 开始就提供了 mutable 的 StringBuffer 类。

**StringBuffer 类对外暴露了可以修改其值的 append、insert、delete 等方法**。一个 StringBuffer 对象在其缓冲区（**一个字符数组 char[]）的容量足够的情况下，调用这些方法可以直接修改 StringBuffer 的值而不必创建新的对象**。（在一个StringBuffer  的缓冲区的容量不足的时候，调用其 append 或者 insert 就会使 StringBuffer 创建一个新的更大的缓冲区，这时则会创建一个新的字符数组 char[] 对象。）

```java
StringBuffer buffer = new StringBuffer(str);
for (int i = 0; i < 1000; i += 1) {
    buffer.append(arr[i]).append(',');
}
str = buffer.toString();
```

**为了实现线程安全，StringBuffer 在 insert、append、delete 这些 public 方法的定义处加了synchronized关键字**

## StringBuilder

**StringBuilder 类是非线程安全的 StringBuffer 类。**

```java
String s = "";
for (int i = 0; i < 1000; i++) {
    s = s + "," + i;
}
```

虽然可以直接拼接字符串，但是，在循环中，每次循环都会创建新的字符串对象，然后扔掉旧的字符串。这样，**绝大部分字符串都是临时对象，不但浪费内存，还会影响GC效率。**

Java标准库提供了`StringBuilder`，它是一个可变对象，可以预分配缓冲区，这样，往`StringBuilder`中新增字符时，不会创建新的临时对象：

```java
StringBuilder sb = new StringBuilder(1024);
for (int i = 0; i < 1000; i++) {
    sb.append(',');
    sb.append(i);
}
String s = sb.toString();
```

支持链式结构

```java
public class Main {
    public static void main(String[] args) {
        var sb = new StringBuilder(1024);
        sb.append("Mr ")
          .append("Bob")
          .append("!")
          .insert(0, "Hello, ");
        System.out.println(sb.toString());
    }
}
```

> 如果我们查看`StringBuilder`的源码，可以发现，进行链式操作的关键是，定义的`append()`方法会返回`this`

对于普通的字符串`+`操作，并不需要我们将其改写为`StringBuilder`，因为Java编译器在编译时就自动把多个连续的`+`操作编码为`StringConcatFactory`的操作。在运行期，`StringConcatFactory`会自动把字符串连接操作优化为数组复制或者`StringBuilder`操作。

## StringJoiner

`StringJoiner`用于构造由定界符分隔的字符序列，并可选择以提供的前缀开始并以提供的后缀结束.

> 需求: 拼接字符串,{"Bob","Alice","Grace"}  => Hello Bob, Alice, Grace!

使用`StringBuider`

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};
        var sb = new StringBuilder();
        sb.append("Hello ");
        for (String name : names) {
            sb.append(name).append(", ");
        }
        // 注意去掉最后的", ":
        sb.delete(sb.length() - 2, sb.length());
        sb.append("!");
        System.out.println(sb.toString());
    }
}
```

使用`StringJoiner`

```java
// 拼接Bob,Alice,Grace
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};
        var sj = new StringJoiner(", ");
        for (String name : names) {
            sj.add(name);
        }
        System.out.println(sj.toString());
    }
}

// 追加前缀和后缀
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};   
        //  new StringJoiner("分割项",前缀,后缀);
        var sj = new StringJoiner(", ", "Hello ", "!");
        for (String name : names) {
            sj.add(name);
        }
        System.out.println(sj.toString());
    }
}
```

`String`还提供了一个静态方法`join()`，这个方法在内部使用了`StringJoiner`来拼接字符串，在不需要指定“开头”和“结尾”的时候，用`String.join()`更方便：

```java
String[] names = {"Bob", "Alice", "Grace"};
var s = String.join(", ", names);
```

## StringBuffer、StringBuilder、StringJoiner的应用场景

1. **StringBuffer**：
   StringBuffer类是一个**线程安全**的可变字符序列，它可以被多个线程安全地修改。主要应用场景包括：
- 字符串的拼接：**由于StringBuffer是可变的，它提供了append()方法来实现字符串的追加，效率较高。在需要频繁拼接字符串且线程安全的场景下，可以使用StringBuffer。**
- 字符串的修改：StringBuffer提供了insert()、delete()、replace()等方法，可以在字符串中进行插入、删除和替换操作。
2. **StringBuilder**：
   StringBuilder类也是一个可变字符序列，与StringBuffer类似，但不**保证线程安全**。主要应用场景包括：
- 字符串的拼接：StringBuilder的append()方法效率较高，适用于需要频繁进行字符串拼接的场景。**由于StringBuilder不是线程安全的，所以在单线程环境下使用效果更好。**
- 字符串的修改：StringBuilder提供了insert()、delete()、replace()等方法，可以对字符串进行插入、删除和替换操作。
3. **StringJoiner**：
   StringJoiner类**用于将一系列字符串拼接成一个字符串，并提供了灵活的分隔符和前后缀设置**。主要应用场景包括：
- 构建以分隔符分隔的字符串：StringJoiner提供了add()方法用于添加字符串，并且在字符串之间自动添加指定的分隔符。这**在构建CSV、SQL语句等需要处理分隔符的场景中非常有用**。
- 前后缀设置：StringJoiner可以设置前缀和后缀，用于包裹拼接后的字符串。

总结：

- **如果需要在多线程环境下进行字符串的拼接和修改操作，应使用StringBuffer。**
- **如果在单线程环境下需要频繁进行字符串的拼接和修改操作，应使用StringBuilder。**
- **如果只需要将一系列字符串拼接成一个字符串，并且需要设置分隔符和前后缀，应使用StringJoiner。**

## 参考

+ [StringJoiner - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1271993169413952)

+ [一文了解 Java 中的 String、StringBuffer 与 StringBuilder](https://www.freecodecamp.org/chinese/news/java-string-stringbuffer-and-stringbuilder/)
