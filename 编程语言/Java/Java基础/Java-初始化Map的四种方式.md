
# 1. java8新特性–双括号初始化
```java
HashMap<String, String > myMap  = new HashMap<String, String>(){{  
      put("a","b");  
      put("b","b");       
}};  
```


# 2.最常见的方式(新建Map对象)
```java
public class Demo{  
    private static final Map<String, String> myMap = new HashMap<String, String>();  
    static{
    	myMap.put("a", "b");  
    	myMap.put("c", "d"); 
    }  
}  
```

<!-- more -->

# 3. java9新特性–最简便的方式
```java
//不可变集合
Map.of("Hello", 1, "World", 2);
```

# 4. Guava
```java
// 最多初始化5个
Map<String, Integer> myMap = ImmutableMap.of("a", 1, "b", 2, "c", 3); 
```
