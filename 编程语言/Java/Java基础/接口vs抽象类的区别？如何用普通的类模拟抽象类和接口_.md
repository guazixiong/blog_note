> [接口vs抽象类的区别？如何用普通的类模拟抽象类和接口？-腾讯云开发者社区-腾讯云]([https://cloud.tencent.com/developer/article/1775611)](https://cloud.tencent.com/developer/article/1775611))

在面向对象编程中，抽象类和接口是两个经常被用到的语法概念，是面向对象四大特性，以及很多设计模式、设计思想、设计原则编程实现的基础。比如，我们可以使用接口来实现面向对象的抽象特性、多态特性和基于接口而非实现的设计原则，使用抽象类来实现面向对象的继承特性和模板设计模式等等。<br />不过，并不是所有的面向对象编程语言都支持这两个语法概念，比如，C++ 这种编程语言只支持抽象类，不支持接口；而像 Python 这样的动态编程语言，既不支持抽象类，也不支持接口。尽管有些编程语言没有提供现成的语法来支持接口和抽象类，我们仍然可以通过一些手段来模拟实现这两个语法概念。<br />这两个语法概念不仅在工作中经常会被用到，在面试中也经常被提及。比如，“接口和抽象类的区别是什么？什么时候用接口？什么时候用抽象类？抽象类和接口存在的意义是什么？能解决哪些编程问题？”等等。<br />你可以先试着回答一下，刚刚我提出的几个问题。如果你对某些问题还有些模糊不清，那也没关系，今天，我会带你把这几个问题彻底搞清楚。下面我们就一起来看！
# In Java, we have two types of relationship:
> 1. **Is-A relationship:** Whenever one class inherits another class, it is called an IS-A relationship.
> 2. **Has-A relationship: **Whenever an instance of one class is used in another class, it is called HAS-A relationship.
> 
在Java中，我们有两种类型的关系：
> 1. Is-A关系：每当一个类继承另一个类时，就称为IS-A关系。
> 2. Has-A关系：每当一个类的实例在另一个类中使用时，就称为HAS-A关系。

<a name="OTJvv"></a>
# 一、什么是抽象类和接口？区别在哪里？
不同的编程语言对接口和抽象类的定义方式可能有些差别，但差别并不会很大。Java 这种编程语言，既支持抽象类，也支持接口，所以，为了让你对这两个语法概念有比较直观的认识，我们拿 Java 这种编程语言来举例讲解。<br />首先，我们来看一下，**在 Java 这种编程语言中，我们是如何定义抽象类的**。<br />下面这段代码是一个比较典型的抽象类的使用场景（模板设计模式）。<br />`Logger` 是一个记录日志的抽象类，`FileLogger` 和 `MessageQueueLogger` 继承 `Logger`，分别实现两种不同的日志记录方式：记录日志到文件中和记录日志到[消息队列](https://cloud.tencent.com/product/cmq?from_column=20065&from=20065)中。`FileLogger` 和 `MessageQueueLogger` 两个子类复用了父类`Logger` 中的 `name`、`enabled`、`minPermittedLevel` 属性和`log()`方法，但因为这两个子类写日志的方式不同，它们又各自重写了父类中的 `doLog()`方法。
```java
// 抽象类
public abstract class Logger {
	private String name;
	private boolean enabled;
	private Level minPermittedLevel;
	
	public Logger(String name, boolean enabled, Level minPermittedLevel) {
		this.name = name;
		this.enabled = enabled;
		this.minPermittedLevel = minPermittedLevel;
	}
	
	public void log(Level level, String message) {
		boolean loggable = enabled && (minPermittedLevel.intValue() <= level.int
		if (!loggable) return;
		doLog(level, message);
	}
	
	protected abstract void doLog(Level level, String message);
}

// 抽象类的子类：输出日志到文件
public class FileLogger extends Logger {
	private Writer fileWriter;
	
	public FileLogger(String name, boolean enabled,
		Level minPermittedLevel, String filepath) {
		super(name, enabled, minPermittedLevel);
		this.fileWriter = new FileWriter(filepath);
	}
	
	@Override
	public void doLog(Level level, String mesage) {
		// 格式化 level 和 message, 输出到日志文件
		fileWriter.write(...);
	}
}

// 抽象类的子类: 输出日志到消息中间件 (比如 kafka)
public class MessageQueueLogger extends Logger {
	private MessageQueueClient msgQueueClient;
	
	public MessageQueueLogger(String name, boolean enabled,
		Level minPermittedLevel, MessageQueueClient msgQueueClient) {
		super(name, enabled, minPermittedLevel);
		this.msgQueueClient = msgQueueClient;
	}
	
	@Override
	protected void doLog(Level level, String mesage) {
		// 格式化 level 和 message, 输出到消息中间件
		msgQueueClient.send(...);
	}
}
```
通过上面的这个例子，我们来看一下，抽象类具有哪些特性。我总结了下面三点。

- **抽象类不允许被实例化，只能被继承**。也就是说，你不能 new 一个抽象类的对象出来（`Logger logger = new Logger(…);` 会报编译错误）。
- 抽象类可以包含属性和方法。方法既可以包含代码实现（比如 `Logger` 中的 `log()` 方法），也可以不包含代码实现（比如 `Logger` 中的 `doLog()` 方法）。**不包含代码实现的方法叫作抽象方法**。
- 子类继承抽象类，必须实现抽象类中的所有抽象方法。对应到例子代码中就是，所有继承 `Logger` 抽象类的子类，都必须重写 `doLog()` 方法。

刚刚我们讲了如何定义抽象类，现在我们再来看一下，**在 Java 这种编程语言中，我们如何定义接口。**
```java
// 接口
public interface Filter {
	void doFilter(RpcRequest req) throws RpcException;
}

// 接口实现类：鉴权过滤器
public class AuthencationFilter implements Filter {
	@Override
	public void doFilter(RpcRequest req) throws RpcException {
		//... 鉴权逻辑..
	}
}

// 接口实现类：限流过滤器
public class RateLimitFilter implements Filter {
	@Override
	public void doFilter(RpcRequest req) throws RpcException {
		//... 限流逻辑...
	}
}

// 过滤器使用 demo
public class Application {
	// filters.add(new AuthencationFilter());
	// filters.add(new RateLimitFilter());
	private List<Filter> filters = new ArrayList<>();
	
	public void handleRpcRequest(RpcRequest req) {
		try {
			for (Filter filter : fitlers) {
				filter.doFilter(req);
			}
		} catch(RpcException e) {
			// ... 处理过滤结果...
		}
		// ... 省略其他处理逻辑...
	}
}
```
上面这段代码是一个比较典型的接口的使用场景。我们通过 Java 中的 interface 关键字定义了一个 Filter 接口。AuthencationFilter 和 RateLimitFilter 是接口的两个实现类，分别实现了对 RPC 请求鉴权和限流的过滤功能。<br />代码非常简洁。结合代码，我们再来看一下，接口都有哪些特性。我也总结了三点。

- **接口不能包含属性（也就是成员变量）。**
- **接口只能声明方法，方法不能包含代码实现。**
- **类实现接口的时候，必须实现接口中声明的所有方法。**

前面我们讲了抽象类和接口的定义，以及各自的语法特性。**从语法特性上对比，这两者有比较大的区别，比如抽象类中可以定义属性、方法的实现，而接口中不能定义属性，方法也不能包含代码实现等等**。除了语法特性，从设计的角度，两者也有比较大的区别。<br />**抽象类实际上就是类，只不过是一种特殊的类，这种类不能被实例化为对象，只能被子类继承。我们知道，继承关系是一种 is-a 的关系，那抽象类既然属于类，也表示一种 is-a 的关系。相对于抽象类的 is-a 关系来说，接口表示一种 has-a 关系，表示具有某些功能**。对于接口，有一个更加形象的叫法，那就是协议（contract）。
<a name="fzTp2"></a>
# 二、抽象类和接口能解决什么编程问题？
刚刚我们学习了抽象类和接口的定义和区别，现在我们再来学习一下，抽象类和接口存在的意义，让你知其然知其所以然。<br />首先，我们来看一下，我们**为什么需要抽象类？它能够解决什么编程问题？**<br />刚刚我们讲到，抽象类不能实例化，只能被继承。而前面的章节中，我们还讲到，继承能解决代码复用的问题。所以，抽象类也是为代码复用而生的。多个子类可以继承抽象类中定义的属性和方法，避免在子类中，重复编写相同的代码。<br />不过，既然继承本身就能达到代码复用的目的，而继承也并不要求父类一定是抽象类，那我们不使用抽象类，照样也可以实现继承和复用。从这个角度上来讲，我们貌似并不需要抽象类这种语法呀。那抽象类除了解决代码复用的问题，还有什么其他存在的意义吗？<br />我们还是拿之前那个打印日志的例子来讲解。我们先对上面的代码做下改造。在改造之后的代码中，`Logger` 不再是抽象类，只是一个普通的父类，删除了 `Logger` 中 `log()`、`doLog()` 方法，新增了 `isLoggable()` 方法。`FileLogger` 和 `MessageQueueLogger` 还是继承 `Logger` 父类，以达到代码复用的目的。具体的代码如下：
```java
// 父类：非抽象类，就是普通的类. 删除了 log(),doLog()，新增了 isLoggable().
public class Logger {
	private String name;
	private boolean enabled;
	private Level minPermittedLevel;
	
	public Logger(String name, boolean enabled, Level minPermittedLevel) {
		//... 构造函数不变，代码省略...
	}
	
	protected boolean isLoggable() {
		boolean loggable = enabled && (minPermittedLevel.intValue() <= level.int
		return loggable;
	}
}

// 子类：输出日志到文件
public class FileLogger extends Logger {
	private Writer fileWriter;
	
	public FileLogger(String name, boolean enabled,
		Level minPermittedLevel, String filepath) {
		//... 构造函数不变，代码省略...
	}
	
	public void log(Level level, String mesage) {
		if (!isLoggable()) return;
		// 格式化 level 和 message, 输出到日志文件
		fileWriter.write(...);
	}
}

// 子类: 输出日志到消息中间件 (比如 kafka)
public class MessageQueueLogger extends Logger {
	private MessageQueueClient msgQueueClient;
	
	public MessageQueueLogger(String name, boolean enabled,
		Level minPermittedLevel, MessageQueueClient msgQueueClient) {
		//... 构造函数不变，代码省略...
	}

	public void log(Level level, String mesage) {
		if (!isLoggable()) return;
		// 格式化 level 和 message, 输出到消息中间件
		msgQueueClient.send(...);
	}
}
```
这个设计思路虽然达到了代码复用的目的，但是无法使用多态特性了。像下面这样编写代码，就会出现编译错误，因为 `Logger` 中并没有定义 `log()` 方法。
```java
Logger logger = new FileLogger("access-log", true, Level.WARN, "/users/wangz
logger.log(Level.ERROR, "This is a test log message.");
```
你可能会说，这个问题解决起来很简单啊。我们在 Logger 父类中，定义一个空的 `log()`方法，让子类重写父类的 `log()` 方法，实现自己的记录日志的逻辑，不就可以了吗？
```java
public class Logger {
	// ... 省略部分代码...
	public void log(Level level, String mesage) { // do nothing... }
}

public class FileLogger extends Logger {
	// ... 省略部分代码...
	@Override
	public void log(Level level, String mesage) {
		if (!isLoggable()) return;
		// 格式化 level 和 message, 输出到日志文件
		fileWriter.write(...);
	}
}

public class MessageQueueLogger extends Logger {
	// ... 省略部分代码...
	@Override
	public void log(Level level, String mesage) {
		if (!isLoggable()) return;
		// 格式化 level 和 message, 输出到消息中间件
		msgQueueClient.send(...);
	}
}
```
这个设计思路能用，但是，它显然没有之前通过抽象类的实现思路优雅。我为什么这么说呢？主要有以下几点原因。

- 在 `Logger` 中定义一个空的方法，会影响代码的可读性。如果我们不熟悉 `Logger` 背后的设计思想，代码注释又不怎么给力，我们在阅读 Logger 代码的时候，就可能对为什么定义一个空的 `log()` 方法而感到疑惑，需要查看 `Logger`、`FileLogger`、`MessageQueueLogger` 之间的继承关系，才能弄明白其设计意图。
- 当创建一个新的子类继承 `Logger` 父类的时候，我们有可能会忘记重新实现 `log()` 方法。之前基于抽象类的设计思路，编译器会强制要求子类重写 `log()` 方法，否则会报编译错误。你可能会说，我既然要定义一个新的 `Logger` 子类，怎么会忘记重新实现`log()`方法呢？我们举的例子比较简单，`Logger` 中的方法不多，代码行数也很少。但是，如果`Logger` 有几百行，有 n 多方法，除非你对 `Logger` 的设计非常熟悉，否则忘记重新实现 `log()`方法，也不是不可能的。
- Logger 可以被实例化，换句话说，我们可以 new 一个 `Logger` 出来，并且调用空的`log()`方法。这也增加了类被误用的风险。当然，这个问题可以通过设置私有的构造函数的方式来解决。不过，显然没有通过抽象类来的优雅。

其次，我们再来看一下，我们**为什么需要接口？它能够解决什么编程问题？**<br />抽象类更多的是为了代码复用，而接口就更侧重于解耦。接口是对行为的一种抽象，相当于一组协议或者契约，你可以联想类比一下 API 接口。调用者只需要关注抽象的接口，不需要了解具体的实现，具体的实现代码对调用者透明。接口实现了约定和实现相分离，可以降低代码间的耦合性，提高代码的可扩展性。<br />实际上，接口是一个比抽象类应用更加广泛、更加重要的知识点。比如，我们经常提到的“基于接口而非实现编程”，就是一条几乎天天会用到，并且能极大地提高代码的灵活性、扩展性的设计思想。关于接口这个知识点，我会单独再用一节课的时间，更加详细全面的讲解，这里就不展开了。
<a name="H6C4w"></a>
# 三、如何模拟抽象类和接口两个语法概念？
在前面举的例子中，我们使用 Java 的接口语法实现了一个 `Filter` 过滤器。不过，如果你熟悉的是 C++ 这种编程语言，你可能会说，C++ 只有抽象类，并没有接口，那从代码实现的角度上来说，是不是就无法实现 `Filter` 的设计思路了呢？<br />实际上，我们可以通过抽象类来模拟接口。怎么来模拟呢？这是一个不错的面试题，你可以先思考一下，然后再来看我的讲解。<br />我们先来回忆一下接口的定义：接口中没有成员变量，只有方法声明，没有方法实现，实现接口的类必须实现接口中的所有方法。只要满足这样几点，从设计的角度上来说，我们就可以把它叫作接口。实际上，要满足接口的这些语法特性并不难。在下面这段C++ 代码中，我们就用抽象类模拟了一个接口（下面这段代码实际上是策略模式中的一段代码）。
```java
class Strategy { // 用抽象类模拟接口
	public:
		~Strategy();
		virtual void algorithm()=0;
	protected:
		Strategy();
};
```
抽象类 Strategy 没有定义任何属性，并且所有的方法都声明为 virtual 类型（等同于Java 中的 abstract 关键字），这样，所有的方法都不能有代码实现，并且所有继承这个抽象类的子类，都要实现这些方法。从语法特性上来看，这个抽象类就相当于一个接口。<br />不过，如果你熟悉的既不是 Java，也不是 C++，而是现在比较流行的动态编程语言，比如 Python、Ruby 等，你可能还会有疑问：在这些动态语言中，不仅没有接口的概念，也没有类似 abstract、virtual 这样的关键字来定义抽象类，那该如何实现上面的讲到的Filter、Logger 的设计思路呢？实际上，除了用抽象类来模拟接口之外，我们还可以用普通类来模拟接口。具体的 Java 代码实现如下所示。
```java
public class MockInteface {
	protected MockInteface() {}
	public void funcA() {
		throw new MethodUnSupportedException();
	}
}
```
我们知道类中的方法必须包含实现，这个不符合接口的定义。但是，我们可以让类中的方法抛出 MethodUnSupportedException 异常，来模拟不包含实现的接口，并且能强迫子类在继承这个父类的时候，都去主动实现父类的方法，否则就会在运行时抛出异常。那又如何避免这个类被实例化呢？实际上很简单，我们只需要将这个类的构造函数声明为 protected 访问权限就可以了。<br />刚刚我们讲了如何用抽象类来模拟接口，以及如何用普通类来模拟接口，那如何用普通类来模拟抽象类呢？这个问题留给你自己思考，你可以留言说说你的实现方法。<br />实际上，对于动态编程语言来说，还有一种对接口支持的策略，那就是 duck-typing。我们在上一节课中讲到多态的时候也有讲过，你可以再回忆一下。
<a name="uL8cz"></a>
# 四、如何决定该用抽象类还是接口？
刚刚的讲解可能有些偏理论，现在，我们就从真实项目开发的角度来看一下，在代码设计、编程开发的时候，什么时候该用抽象类？什么时候该用接口？<br />实际上，判断的标准很简单。**如果我们要表示一种 is-a 的关系，并且是为了解决代码复用的问题，我们就用抽象类**；**如果我们要表示一种 has-a 关系，并且是为了解决抽象而非代码复用的问题，那我们就可以使用接口**。<br />从类的继承层次上来看，抽象类是一种自下而上的设计思路，先有子类的代码重复，然后再抽象成上层的父类（也就是抽象类）。而接口正好相反，它是一种自上而下的设计思路。我们在编程的时候，一般都是先设计接口，再去考虑具体的实现。
<a name="tq1Xf"></a>
# 五、抽象类适用场景
java类之间为`is-a`

- 自定义日志Logger
- 自定义过滤器Filter
- 自定义计算工时
- 自定义邮箱通知
- 自定义短信通知
- ........
