# 微信支付对接V3 JSAPI操作手册

## 前提准备

+ Java 1.8+。

+ [成为微信支付商户](https://pay.weixin.qq.com/index.php/apply/applyment_home/guide_normal)。

+ [商户 API 证书](https://wechatpay-api.gitbook.io/wechatpay-api-v3/ren-zheng/zheng-shu#shang-hu-api-zheng-shu)：指由商户申请的，包含商户的商户号、公司名称、公钥信息的证书。
- [商户 API 私钥](https://wechatpay-api.gitbook.io/wechatpay-api-v3/ren-zheng/zheng-shu#shang-hu-api-si-yao)：商户申请商户API证书时，会生成商户私钥，并保存在本地证书文件夹的文件 apiclient_key.pem 中。
- [APIv3 密钥](https://wechatpay-api.gitbook.io/wechatpay-api-v3/ren-zheng/api-v3-mi-yao)：为了保证安全性，微信支付在回调通知和平台证书下载接口中，对关键信息进行了 AES-256-GCM 加密。APIv3 密钥是加密时使用的对称密钥。

| 参数名称     | 参数说明                                                                                                                         |
| -------- | ---------------------------------------------------------------------------------------------------------------------------- |
| AppID    | 商户应用载体的AppID，可以是公众号，小程序或App                                                                                                  |
| mchid    | 商户在微信侧申请入驻的收款账号                                                                                                              |
| API v3密钥 | 商户在商户平台设置的API v3密钥，主要用于对敏感字段信息的加密或解密，具体设置流程请参考各产品接入前准备说明                                                                     |
| 商户API证书  | 商户在商户平台下载的证书，主要用于API请求的签名生成及验证，具体下载操作说明请参考各产品接入前准备说明                                                                         |
| OpenID   | 用户在直连商户应用下的用户标示<br>[OpenID获取详见](https://pay.weixin.qq.com/wiki/doc/apiv3_partner/terms_definition/chapter1_1_3.shtml#part-3) |

## 基于SDK开发

[SDK，工具 | 微信支付商户平台文档中心](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay6_0.shtml)

### 注意事项:

1. 一个mchId在项目运行期间只允许build一次(需要采用单例模式)

> 每个商户号只能创建一个 `RSAAutoCertificateConfig`。同一个商户号构造多个实例，会抛出 `IllegalStateException` 异常。
> 
> 我们建议你将配置类作为全局变量。如果你的程序是多线程，建议使用**多线程安全**的单例模式。

```java
/**
 * @author: pangdi
 * @Date: 2023/5/9 15:28
 * @Description: WxRsaConfigSingleton是一个单例类，用于创建和管理微信支付RSA配置对象。
 */
@Component
public class WxRsaConfigSingleton {

    /**
     * 单例实例
     */
    private static volatile WxRsaConfigSingleton instance;

    /**
     * 商户配置对象Map，使用ConcurrentHashMap实现线程安全
     */
    private final Map<String, Config> configMap = new ConcurrentHashMap<>();

    /**
     * 私有构造方法，用于禁止外部直接实例化。
     */
    private WxRsaConfigSingleton() {}

    /**
     * 获取WxRsaConfigSingleton的单例实例。
     *
     * @return WxRsaConfigSingleton的单例实例
     */
    public static WxRsaConfigSingleton getInstance() {
        if (instance == null) {
            synchronized (WxRsaConfigSingleton.class) {
                if (instance == null) {
                    instance = new WxRsaConfigSingleton();
                }
            }
        }
        return instance;
    }

    /**
     * 获取指定商户的RSA配置对象，如果不存在则创建一个新的。
     *
     * @param merchantsManager 本地商户管理对象
     * @return 商户的RSA配置对象
     */
    public Config getWxRsaConfig(saConfig(String mchId,String privateKeyFromPath,String merchantSerialNumber,String apiV3Key) {
        String mchId = merchantsManager.getMchId();
        Config config = configMap.get(mchId);
        if (config == null) {
            synchronized (WxRsaConfigSingleton.class) {
                config = configMap.get(mchId);
                if (config == null) {
                    // 创建新的商户RSA配置对象
                    config = createWxRsaConfig(mchId,privateKeyFromPath,merchantSerialNumber,apiV3Key);
                    configMap.put(mchId, config);
                }
            }
        }
        return config;
    }

    /**
     * 根据指定的商户管理对象创建一个新的RSA配置对象。
     * 
     * @param mchId  商户号
     * @param privateKeyFromPath 商户API私钥路径
     * @param merchantSerialNumber 商户证书序列号
     * @param apiV3Key apiV3秘钥
     * @return 商户的RSA配置对象
     */
    private Config createWxRsaConfig(String mchId,String privateKeyFromPath,String merchantSerialNumber,String apiV3Key) {
        // 自动更新平台证书RSA配置
        return new RSAAutoCertificateConfig.Builder()
                // 商户号
                .merchantId(mchId)
                // 商户API私钥路径(api证书路径 *.pem文件)
                .privateKeyFromPath(privateKeyFromPath)
                // 商户证书序列号
                .merchantSerialNumber(merchantSerialNumber)
                // apiV3秘钥
                .apiV3Key(apiV3Key)
                // 平台证书直接通过微信自动生成
                .build();
    }


}


```

