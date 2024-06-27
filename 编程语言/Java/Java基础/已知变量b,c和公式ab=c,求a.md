```java
public class Main {
    public static void main(String[] args) {
        int b = 5;
        int c = 2;
        int a = getNumber(b, c);
        System.out.println("a是：" + a);
    }

    // 整除
    public static int getNumber(int b, int c) {
        return c / b;
    }

    // 向上取整
    public static int getNumber(int b, int c) {
        return (c + b - 1) / b;
    }

    // 向下取整
    public static int getNumber(int b, int c) {
        return (c + b - 1) / b - 1;
    }
}

```
如果能保证a,b均为整数,可以直接使用c/b来获取a值,在不确定的情况下,`c/b`可能为浮点数,以向下取整为例,使用 `(c + b - 1) / b - 1` 这个表达式是向下取整的。  在这个表达式中，我们通过将 `c` 加上 `b` 减 `1`，然后除以 `b`，最后再减去 `1`，来确保结果是整数并且向下取整。  向下取整是一种舍弃小数部分并且返回最大整数的操作。在这个表达式中，向下取整确保了结果不会大于真实的a，并且始终是一个整数。
