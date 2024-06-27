| 方法 | 描述 |
| --- | --- |
| notNull(Object object) | 当 object 不为 null 时抛出异常，notNull(Object object, String message) 方法允许您通过 message 定制异常信息。和 notNull() 方法断言规则相反的方法是 isNull(Object object)/isNull(Object object, String message)，它要求入参一定是 null； |  |
| isTrue(boolean expression) | 当 expression 不为 true 抛出异常； |  |
| isTrue(boolean expression, String message) |  |
| notEmpty(Collection collection)  | 当集合未包含元素时抛出异常。
　　notEmpty(Map map) / notEmpty(Map map, String message) 和 notEmpty(Object[] array, String message) / notEmpty(Object[] array, String message) 分别对 Map 和 Object[] 类型的入参进行判断； |  |
| notEmpty(Collection collection, String message) |  |
| hasLength(String text) | 当 text 为 null 或长度为 0 时抛出异常； |  |
| hasLength(String text, String message) |  |
| hasText(String text)  | text 不能为 null 且必须至少包含一个非空格的字符，否则抛出异常； |  |
| hasText(String text, String message) |  |
| isInstanceOf(Class clazz, Object obj) | 如果 obj 不能被正确造型为 clazz 指定的类将抛出异常； |  |
| isInstanceOf(Class type, Object obj, String message) |  |
| isAssignable(Class superType, Class subType) | subType 必须可以按类型匹配于 superType，否则将抛出异常； |  |
| isAssignable(Class superType, Class subType, String message) |  |
