# SHA256WithRSA 加密工具类

```java
import cn.hutool.core.codec.Base64;

import java.nio.charset.StandardCharsets;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.Signature;
import java.security.spec.PKCS8EncodedKeySpec;

/**
 * @author: pangdi
 * @Date: 2023/6/1 14:11
 * @Description: rea加密工具类
 */
public class RsaUtil {

    /**
     * rsa算法加密.
     *
     * @param data 数据源
     * @param privateKey 秘钥
     * @return 加密数据源
     * @throws Exception rsa算法加密数据源时,产生异常,向上抛出
     */
    public static String sign(String data, String privateKey) throws Exception {
        // 解码私钥
        byte[] privateKeyBytes = null;
        // 判断私钥是否经过Base64编码
        if (isBase64Encoded(privateKey)) {
            privateKeyBytes = Base64.decode(privateKey);
        } else {
            privateKeyBytes = privateKey.getBytes(StandardCharsets.UTF_8);
        }

        // 构造私钥对象
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PrivateKey generatePrivate = keyFactory.generatePrivate(keySpec);

        // 使用SHA256WithRSA算法进行签名
        Signature signature = Signature.getInstance("SHA256WithRSA");
        signature.initSign(generatePrivate);
        signature.update(data.getBytes("UTF-8"));

        // 生成签名结果
        byte[] signBytes = signature.sign();

        // 进行Base64编码
        return Base64.encode(signBytes);
    }

    private static boolean isBase64Encoded(String input) {
        try {
            Base64.decode(input);
            return true;
        } catch (IllegalArgumentException e) {
            return false;
        }
    }
}
```

