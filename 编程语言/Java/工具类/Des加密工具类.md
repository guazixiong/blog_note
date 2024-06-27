# Des加密工具类

```java
import cn.hutool.core.util.StrUtil;
import org.springframework.stereotype.Component;
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

/**
 * Des加密工具类.
 */
@Component
public class DesUtil {
    private static final String IV = "union968";

    /**
     * 加密算法.
     *
     * @param src 数据源
     * @param key 秘钥,长度必须是8的倍数
     * @return 加密后的数据
     * @throws Exception
     */
    public static String nonNullEncryptDESCBC(final String src, final String key) throws Exception{
        if (!StrUtil.isEmpty(src)) {
            return encryptDESCBC(src,key);
        }
        return null;
    }

    /**
     * 加密算法.
     *
     * @param src 数据源
     * @param key 秘钥,长度必须是8的倍数
     * @return 加密后的数据
     * @throws Exception
     */
    public static String encryptDESCBC(final String src, final String key) throws Exception{
        // 生成key,同时制定是des还是DESede,两者的key长度要求不同.
        final DESKeySpec desKeySpec = new DESKeySpec(key.getBytes(StandardCharsets.UTF_8));
        final SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        final SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
        // 加密向量
        final IvParameterSpec iv = new IvParameterSpec(IV.getBytes(StandardCharsets.UTF_8));
        final Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE,secretKey,iv);
        // 通过base64,将加密数组转换成字符串
        final byte[] b = cipher.doFinal(src.getBytes(StandardCharsets.UTF_8));
        return Base64.getEncoder().encodeToString(b);
    }

    /**
     * des 解密.
     *
     * @param src 数据源
     * @param key 秘钥
     * @return 解密数据
     * @throws Exception
     */
    public static String decryptDESCBC(final String src, final String key) throws Exception {
        // 通过base64,将字符串转换成byte数组
        final byte[] bytes = Base64.getDecoder().decode(src);
        // 解密key
        final DESKeySpec desKeySpec = new DESKeySpec(key.getBytes(StandardCharsets.UTF_8));
        final SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        final SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
        // 向量
        final IvParameterSpec iv = new IvParameterSpec(IV.getBytes(StandardCharsets.UTF_8));
        final Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE,secretKey,iv);
        final byte[] retByte = cipher.doFinal(bytes);
        return new String(retByte);
    }
}

```

