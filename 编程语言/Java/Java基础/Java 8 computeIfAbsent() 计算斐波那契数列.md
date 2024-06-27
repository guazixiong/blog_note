# Java 8 computeIfAbsent() 计算斐波那契数列

```java
import java.util.Map;
import java.util.HashMap;

public class Fibonacci {
    private Map<Integer, Integer> memo = new HashMap<>();

    public int fib(int n) {
        if (n < 0) {
            throw new IllegalArgumentException("n must be greater than or equal to zero");
        }

        if (n == 0) {
            return 0;
        }

        if (n == 1) {
            return 1;
        }

        return memo.computeIfAbsent(n, k -> fib(k - 1) + fib(k - 2));
    }
}
```
