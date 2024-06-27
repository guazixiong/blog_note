# Amdahl's law（阿姆达尔定律）公式推导与思考

> 原文链接: Amdahl's law（阿姆达尔定律）公式推导与思考 - 简书
> https://www.jianshu.com/p/3ff6c95b4ae6

## 1. 介绍

*Amdahl's law（阿姆达尔定律）* 由计算机科学家 Gene Amdahl  在 1967 年提出，旨在用公式描述在并行计算中，多核处理器理论上能够提高多少倍速度，公式如下：

![](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271753546.png)
 ![S](https://math.jianshu.com/math?formula=S) 为 speedup，代表全局加速倍速（原来总时间/ 加速后总时间），![a](https://math.jianshu.com/math?formula=a) 为并行计算所占比例（可以并行计算代码量 / 总代码量），![n](https://math.jianshu.com/math?formula=n) 为并行节点处理个数，可以理解为 CPU 的核心数，这里先简要介绍下，后面会详细说明并且推导。

## 2. 前置说明

### 2.1 Fraction enhanced

**Fraction enhanced** 顾名思义是部分提高。例如我的程序总共有 100 行代码，其中 50 行是可以通过并行计算的，那么这 50 行代码就是 **Fraction enhanced** 。但是实际上 **Fraction enhanced** 是一个比例数值，是**并行计算代码 / 总代码量**。
![image-20230227175403012](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271754081.png)
 例如上面的例子，![Fraction\ enhanced = 50/100=0.5](https://math.jianshu.com/math?formula=Fraction%5C%20enhanced%20%3D%2050%2F100%3D0.5) ，由此我们可以发现，**Fraction enhanced** 的值*永远小于 1*。

### 2.2 Speedup enhanced

![image-20230227175426870](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271754910.png)

如上面公式所得，**Speedup enhanced** 等于 **原有运行时间 / 并行计算加速后的时间** 。

例如系统原来串行计算需要 6 秒，加速后只需要 3 秒，那么 ![Speed\ enhanced=6/3=2](https://math.jianshu.com/math?formula=Speed%5C%20enhanced%3D6%2F3%3D2) 。

由此可知 **Speedup enhanced** 的值*永远大于 1*。

### 2.3 带入 Amdahl's law

我们分别把 **Fraction enhanced** 和 **Speedup enhanced** 带入 *Amdahl's law*。

**Fraction enhanced** 对应公式中的 ![a](https://math.jianshu.com/math?formula=a) ，即并行计算所占比例。**Speedup enhanced** 对应 ![n](https://math.jianshu.com/math?formula=n) ，即并行节点处理个数。

**Speedup enhanced 为什么可以代替 ![n](https://math.jianshu.com/math?formula=n) ？**

这里大家可能有一点疑问，Speedup enhanced 明明是 未加速前时间 / 加速后的时间，为什么就可以代表并行节点处理个数。在理论上，单核处理器处理一个任务需要 100 秒，那么双核处理它应该需要 50 秒。时间上它提速了 2 倍， cpu 个数上它也提升了 2 倍，故两个可以替换。

![image-20230227175445220](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271754264.png)

带入后公示得：
 ![image-20230227175500526](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271755574.png)

## 3. 证明

- ![S](https://math.jianshu.com/math?formula=S) 为我们所需要的结果，全局提速倍速
- ![Old\ total\ execution\ time](https://math.jianshu.com/math?formula=Old%5C%20total%5C%20execution%5C%20time) （原来系统执行总时间）为 ![T](https://math.jianshu.com/math?formula=T)
- ![New\ total\ execution\ time](https://math.jianshu.com/math?formula=New%5C%20total%5C%20execution%5C%20time) （加速后系统执行总时间）为 ![T'](https://math.jianshu.com/math?formula=T')
- 系统中并行代码块（指能够通过并行计算加速的代码块）原来执行时间为 ![t](https://math.jianshu.com/math?formula=t)
- 系统中并行代码块（指能够通过并行计算加速的代码块）加速后执行时间为 ![t'](https://math.jianshu.com/math?formula=t')
- 系统中串行代码块（指不能通过并行计算加速的代码块）执行时间为 ![t_n](https://math.jianshu.com/math?formula=t_n)
- ![Fraction_{enhanced}](https://math.jianshu.com/math?formula=Fraction_%7Benhanced%7D) 为 ![f'](https://math.jianshu.com/math?formula=f')
- ![Speedup_{enhanced}](https://math.jianshu.com/math?formula=Speedup_%7Benhanced%7D) 为 ![s'](https://math.jianshu.com/math?formula=s')

![image-20230227175517464](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271755510.png)

带入公式得：
![image-20230227175535199](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271755269.png)
 证明完毕！

## 4. 总结

让我们回到最初的公式

![](Amdahl%27s%20law%EF%BC%88%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B%EF%BC%89%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC%E4%B8%8E%E6%80%9D%E8%80%83.assets/202302271753546-170031735489215.png)

 ![a](https://math.jianshu.com/math?formula=a) 为并行计算所占比例，![n](https://math.jianshu.com/math?formula=n) 为并行节点处理个数。

当 $a == 0$ 时（即只有串行没有并行），无论$n$ 为多少，加速比$S$均为1。

当$n->正无穷$,当 cpu 核心数无限增多的时候，极限加速比$S = 1/(1-a)$ 

例如若并行代码有 75%，极限加速比不能超出 4。

**由此我们可知，在并行系统中一味的增加运算资源，并不能永远成倍的提升系统整体性能。**