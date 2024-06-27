# 记leetcode 刷题,回顾运算符优先级

![img](%E8%AE%B0leetcode%20%E5%88%B7%E9%A2%98,%E5%9B%9E%E9%A1%BE%E8%BF%90%E7%AE%97%E7%AC%A6%E4%BC%98%E5%85%88%E7%BA%A7.assets/1701324284835-5163068e-7130-4d86-a68c-0443fc864230.png)

官方解答:

```java
class ChapterSolution {
    public boolean closeStrings(String word1, String word2) {
        int[] count1 = new int[26], count2 = new int[26];
        for (char c : word1.toCharArray()) {
            count1[c - 'a']++;
        }
        for (char c : word2.toCharArray()) {
            count2[c - 'a']++;
        }
        for (int i = 0; i < 26; i++) {
            if (count1[i] > 0 && count2[i] == 0 || count1[i] == 0 && count2[i] > 0) {
                return false;
            }
        }
        Arrays.sort(count1);
        Arrays.sort(count2);
        return Arrays.equals(count1, count2);
    }
}
```

在学习过程中对以下代码产生了疑问,他的执行顺序是什么,

```java
            if (count1[i] > 0 && count2[i] == 0 || count1[i] == 0 && count2[i] > 0) {
                return false;
            }
```

个人理智告诉我,会按照以下执行,那么理论依据是什么,想不通编译器究竟是如何去处理

```java
            if (count1[i] > 0 && count2[i] == 0) || (count1[i] == 0 && count2[i] > 0) {
                return false;
            }
```

查看编译后的字节码文件

```bash
# 生成SynchronizedDemo.class文件
javac ChapterSolution.java
# 会显示class的字节码文件
javap -c -s -v -l ChapterSolution.class
```

```java
// 字节码文件
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)

package com.pbad.leetcode.chapter1657;

import java.util.Arrays;

public class ChapterSolution {
    public ChapterSolution() {
    }

    public static boolean closeStrings(String var0, String var1) {
        int[] var2 = new int[26];
        int[] var3 = new int[26];
        char[] var4 = var0.toCharArray();
        int var5 = var4.length;

        int var6;
        char var7;
        for(var6 = 0; var6 < var5; ++var6) {
            var7 = var4[var6];
            ++var2[var7 - 97];
        }

        var4 = var1.toCharArray();
        var5 = var4.length;

        for(var6 = 0; var6 < var5; ++var6) {
            var7 = var4[var6];
            ++var3[var7 - 97];
        }

        for(int var8 = 0; var8 < 26; ++var8) {
            if (var2[var8] > 0 && var3[var8] > 0 || var2[var8] == 0 && var3[var8] > 0) {
                return false;
            }
        }

        Arrays.sort(var2);
        Arrays.sort(var3);
        return Arrays.equals(var2, var3);
    }
}
```

可以看到字节码的文件中,执行顺序没有,看来问题出在是`&&`和`||`身上了,是他们决定了逻辑执行先后性.

```java
        for(int var8 = 0; var8 < 26; ++var8) {
            if (var2[var8] > 0 && var3[var8] > 0 || var2[var8] == 0 && var3[var8] > 0) {
                return false;
            }
        }
```

浅浅的汇总了一下,

Java语言中提供了很多运算符来操作变量

1. 赋值运算符
2. 算术运算符
3. 关系运算符
4. 逻辑运算符
5. 位运算符
6. 三目运算符
7. `instanceof` 运算符

## 一、赋值运算符

赋值符号 " `=`  "
语法：变量名 = 表达式；
可以和算术运算符结合成复合赋值运算符，例如" `+=`  "、" `-=` "、" `*=` "、" `/=` "、" `%=` "
示例：

```java
int a = 8;

b+= 18;

m *=5; 
```

## 二、算术运算符

| 运算符 | 含义     | 示例       | 结果     |
| ------ | -------- | ---------- | -------- |
| `+`    | 加法运算 | 2+3        | 5        |
| `-`    | 减法运算 | 5-2        | 3        |
| `*`    | 乘法运算 | 2*3        | 6        |
| `/`    | 除法运算 | 8/3        | 2        |
| `%`    | 取余运算 | 5%3        | 2        |
| `++`   | 自增运算 | i=2; j=i++ | i=3; j=2 |
| `--`   | 自减运算 | i=2; j=i-- | i=1; j=2 |

符号 " `+` "、" `-` "、" `*` "、" `/` "、" `%` "、" `++` "、" `--` "
几点注意：

1. 除法运算，两个操作数是整型的，结果也会是整型的，舍弃掉小数部分；如果有一个数是浮点数，结果将自动转型为浮点型
2. 取余运算，两个操作数是整型的，结果也会是整型的，如果有一个数是浮点数，结果将自动转型为浮点型
3. 自增自减运算，a++ 相当于 a = a + 1, a++ 是先运用在计算，++a 先计算在运用

## 三、关系运算符

| 运算符 | 含义     | 示例 | 结果  |
| ------ | -------- | ---- | ----- |
| `==`   | 等于     | 2==3 | false |
| `!=`   | 不等于   | 5!=2 | true  |
| `<`    | 小于     | 2<3  | true  |
| `>`    | 大于     | 8>3  | true  |
| `<=`   | 小于等于 | 5<=3 | false |
| `>=`   | 大于等于 | 5>=3 | true  |

几点注意：

1. " = "是赋值运算，" == "是等于运算
2. " > "、" < "、" >= "、" <= "只支持数值类型的比较，" == "、" != "支持所有数据类型的比较
3. 关系表达式的运算结果是布尔值

## 四、逻辑运算符

| 运算符 | 含义     | 运算规则                                                     |
| ------ | -------- | ------------------------------------------------------------ |
| `&`    | 逻辑与   | 两个操作数都是true，结果才是true；左边取值无论真假，右边都会运算 |
| `|`    | 逻辑或   | 两个操作数一个是true，结果就是true；左边取值无论真假，右边都会运算 |
| `^`    | 逻辑异或 | 两个操作相同，结果是false；两个操作不相同，结果是true        |
| `!`    | 逻辑非   | 操作数是true，结果是false；操作数是false，结果是true         |
| `&&`   | 短路与   | 两个操作数都是true，结果才是true；左边取值是false，右边不会运算 |
| `||`   | 短路或   | 两个操作数一个是true，结果就是true；左边取值是true，右边不会运算 |

几点注意：

1. 操作数只能是布尔型，操作结果也是布尔型
2. `&` 和 `&&` 的区别： `&&` 左边是`false`时，不计算右边的表达式，左假即假； `&` 无论左边真假都会计算右边的表达式

## 五、位运算符

| 运算符 | 含义       | 运算规则                               |
| ------ | ---------- | -------------------------------------- |
| `&`    | 按位与     | 两个操作数都为1，结果为1               |
| `|`    | 按位或     | 两个操作数一个为1，结果为1             |
| `^`    | 按位异或   | 两个操作数相同为0，不同为1             |
| `~`    | 按位取反   | 操作数为1，结果为0；操作数为0，结果为1 |
| `<<`   | 左移       | 右边空位补0                            |
| `>>`   | 右移       | 左边空位补最高位，即符号位             |
| `>>>`  | 无称号右移 | 左边空位补0                            |

示例: a和b是两个整数，下面是按位计算的形式

```java
a = 0011 1100 ;    
b = 0000 1101 ;

a & b = 0000 1100 ;
a | b = 0011 1101 ;
a ^ b = 0011 0001 ;
~a = 1100 0011 ;
a<<2  = 1111 0000 ;
a>>2  = 1111 ;
a>>>2 = 0000 1111 ;
```

## 六、三目运算符

也叫三元运算符，或是条件运算符，是Java语言中唯一需要三个操作数的运算符
符号: `表达式1 ? 表达式2 : 表达式3`
如果表达式1为true，则返回表达式2的值，如果表达式1为false，则返回表达式3的值
示例：

```java
int m,n;
m = 5<10 ? 10 : 20;  //先判断5<7，为真，则m = 10
n = 5>10 ? 10 : 20;  //先判断5<7，为假，则n = 20
```

## 七、`instanceof`运算符

`instanceof`运算符用于操作对象实例，检查该对象是否是一个特定类型（类类型或接口类型），结果返回一个布尔值
示例：

```java
String name = "张三";
boolean flag = name instanceof String;  //name是String类型的，返回true
```

## 运算符号的优先级

| 优先级 | 运算符                                                       | 结合性   |
| ------ | ------------------------------------------------------------ | -------- |
| 1      | `( )` 、`[ ]`、 `.`                                          | 从左到右 |
| 2      | `!` 、 `~`  、`++` 、 `--`                                   | 从右到左 |
| 3      | `*` 、 `/` 、`%`                                             | 从左到右 |
| 4      | `+` 、 `-`                                                   | 从左到右 |
| 5      | `<<` 、 `>>` 、`>>>`                                         | 从左到右 |
| 6      | `<` 、`<=`  、`>`、 `>=`、 `instanceof`                      | 从左到右 |
| 7      | `==`、 `!=`                                                  | 从左到右 |
| 8      | `&`                                                          | 从左到右 |
| 9      | `^`                                                          | 从左到右 |
| 10     | `|`                                                          | 从左到右 |
| 11     | `&&`                                                         | 从左到右 |
| 12     | `||`                                                         | 从左到右 |
| 13     | `?`、 `:`                                                    | 从左到右 |
| 14     | `=`、 `+=`、 `-=`、 `*=`、 `/=`、 `%=`、 `&=`、 `|=`、 `^=`、 `~=`、 `<<=`、 `>>=` `>>>=` | 从右到左 |
| 15     | `，`                                                         | 从右到左 |

当多个运算符出现在一个表达式中，谁的优先级别高，就先执行谁。在一个多运算符的表达式中，运算符优先级不同会导致最后得出的结果完全不一样。
有一个口诀可以帮助记忆：

### 单算移关与，异或逻条赋

**括号级别最高，逗号级别最低，单目 > 算术 > 位移 > 关系 > 逻辑 > 三目 > 赋值。**

回归正文: 

```java
// &&` 为11位,`||`为12位,所以先执行`&&`

class ChapterSolution {
    public boolean closeStrings(String word1, String word2) {
        int[] count1 = new int[26], count2 = new int[26];
        for (char c : word1.toCharArray()) {
            count1[c - 'a']++;
        }
        for (char c : word2.toCharArray()) {
            count2[c - 'a']++;
        }
        for (int i = 0; i < 26; i++) {
            if ((count1[i] > 0 && count2[i] == 0) || (count1[i] == 0 && count2[i] > 0)) {
                return false;
            }
        }
        Arrays.sort(count1);
        Arrays.sort(count2);
        return Arrays.equals(count1, count2);
    }
}
```

先执行`(count1[i] > 0 && count2[i] == 0)`,再执行`(count1[i] == 0 && count2[i] > 0)`,最后执行`||`操作.

- https://www.jianshu.com/p/9d2204712097