# java.security.InvalidKeyException: Illegal key size

## 异常前提

接口测试:

对数据源进行加密,提示长度收到限制

```java
	SecretKeySpec secretKeySpec = new SecretKeySpec(aesKey, "AES");
        Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
        IvParameterSpec ips = createCtrIv(nonce);
        cipher.init(1, secretKeySpec, ips);　　　　//当代码运行到这一行时就报错了。爆出上面的异常
```

## 问题原因

如果密钥大于128, 会抛出`java.security.InvalidKeyException: Illegal key size `异常. 因为**密钥长度是受限制的, java运行时环境读到的是受限的policy文件. 文件位于${java_home}/jre/lib/security**, 这种限制是**因为美国对软件出口的控制**.

解决方案:去官方下载JCE无限制权限策略文件。

### 下载地址

jdk 5: http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-java-plat-419418.html#jce_policy-1.5.0-oth-JPR

jdk6: http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html

JDK7的下载地址: http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
JDK8的下载地址: http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html 

JDK8 https://wwi.lanzoup.com/iXGs404zm1dg

下载后解压，可以看到`local_policy.jar`和`US_export_policy.jar`以及`readme.txt`

复制 `local_policy.jar`和`US_export_policy.jar`

替换 **C:\Program Files (x86)\Java\jdk1.8.0_121\jre\lib\security** 目录下的这两个包即可

> jdk安装目录



