# Java反射将对象中的所有非空请求参数按照key的ASCII顺序进行升序排序

> 通过java反射获取子类对象时,无法获取父类字段,因为请保证需要解析的对象没有继承类

利用反射获取class对象中**所有非父类字段信息**,按照key的ASCII顺序进行升序排序

```java
    private static final Map<String, String> FILTER_MD5_FIELD = new HashMap<>();
    static{
        FILTER_MD5_FIELD.put("filterField1", null);
        FILTER_MD5_FIELD.put("filterField2", null);
    }

	/**
     * 将对象中的所有非空请求参数按照key的ASCII顺序进行升序排序，
     * 然后将它们拼接成key=value&key=value的格式。
     *
     * @param obj 包含请求参数的对象
     * @return 按照key排序并拼接成字符串的请求参数
     */
    private static String sortAndConcatenateParams(Object obj) {
        Map<String, Object> paramMap = new TreeMap<>();
        // 将对象中所有非空请求参数放入map中
        for (java.lang.reflect.Field field : obj.getClass().getDeclaredFields()) {
            field.setAccessible(true);
            try {
                Object value = field.get(obj);
                if (value != null && !value.toString().isEmpty() && !StrUtil.isEmpty(value.toString())) {
                    paramMap.put(field.getName(), value);
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
        // 将map按照key进行升序排序，并拼接成字符串
        StringBuilder sb = new StringBuilder();
        paramMap.keySet().forEach(key->{
            // 过滤不需要的字段
            if (!FILTER_MD5_FIELD.containsKey(key)) {
                if (sb.length() > 0) {
                    sb.append("&");
                }
                sb.append(key).append("=").append(paramMap.get(key));
            }
        });
        return sb.toString();
    }
```

```java
User user = new User();
String sortAndConcatenateParams = sortAndConcatenateParams(user);
```

