```java
public class Main {
    public static void main(String[] args) {
        int num1 = 10;
        int num2 = 20;
        int middle = getMiddleNumber(num1, num2);
        System.out.println("中间数是：" + middle);
    }

    // 获取两个数之间的中间数
    public static int getMiddleNumber(int a, int b) {
        // 使用位运算的方式，可以避免整数溢出的问题
        return (a + b) / 2;
    }

    // 获取两个数之间的中间数
    public static int getMiddleNumber(int a, int b) {
        // 使用位运算的方式，可以避免整数溢出的问题
        return a + (b - a) / 2;
    }
}

```
在这个例子中，**getMiddleNumber** 方法使用了 **(b - a) / 2** 的方式计算两个数的中间数。这种方式可以避免整数溢出的问题，并且在处理大数值时更为安全。如果你知道两个数之间的范围不会导致溢出，也可以使用简单的 **(a + b) / 2**。

