# Vanblog + PicGo + 腾讯云COS搭建博客

[toc]

## 安装VanBlog

**此处直接采用VanBlog提供的脚本部署(一键化部署)**

```bash
# 一键部署
curl -L https://vanblog.mereith.com/vanblog.sh -o vanblog.sh && chmod +x vanblog.sh && ./vanblog.sh

# 输入数字1后,填写相关邮箱地址 
1
```
![image-20231119164228841](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164228841.png)

- 浏览器访问`http://ip地址/admin/init`,按照提示进行初始化,具体设置项参考
## 安装与配置PicGo
下载地址：[https://github.com/Molunerfinn/PicGo/releases](https://github.com/Molunerfinn/PicGo/releases)

![image-20231119164300075](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164300075.png)

## 腾讯云配置COS
访问链接: 

- **购买对象存储COS**

![image-20231119164321478](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164321478.png)

- **配置COS**
### 创建存储桶
进入【控制台】->【对象存储】→【存储桶列表】→【创建存储桶】

![image-20231119164340588](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164340588.png)



> 注意此时的`名称`和`所属区域`,最后配置PicGo时,与其有关
> 如: ap-区域拼音
> 中国-重庆,对应的存储区域为`ap-chongqing`
> 中国-南京,对应的存储区域为`ap-nanjing`

### 密钥配置
**进入【控制台】->【密钥管理】→【新建密钥】**

![image-20231119164355729](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164355729.png)

**打开PicGo软件,点击【腾讯云COS】，填入相关信息**

![image-20231119164406801](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164406801.png)

| **填写项** | **数据来源** | **备注** |
| --- | --- | --- |
| 图床配置名 | 随便填写 |  |
| COS版本 | V5 |  |
| 设定SecretId | 腾讯云API密钥的SecretId |  |
| 设定SecretKey | 腾讯云API密钥的SecretKey |  |
| 设定Bucket | 腾讯云存储桶自定义名称 |  |
| 设定AppId | 腾讯云API密钥的AppId |  |
| 设定存储区域 | 腾讯云存储桶所属区域 | 中国南京,ap-nanjing,中国重庆,ap-重庆 |
| 设定存储路径 | 腾讯云存储桶文件路径 | 可以为空,也可以自定义存储桶文件列表路径 |
| 设定自定义域名 | 上传图片后自动拼接的域名 | 可以为空 |
| 设定网址后缀 | 上传图片后自动拼接的网址后缀 | 可以为空 |

**最后,配置项,设置为默认图床,否则PicGO上传时,不会默认走腾讯云COS**

## PicGo相关配置项
打开PicGo,进入【PicGo设置】，为方便使用，可以将以下几项勾选

![image-20231119164422697](Vanblog%20+%20PicGo%20+%20%E8%85%BE%E8%AE%AF%E4%BA%91COS%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2.assets/image-20231119164422697.png)

## 参考链接:

1. VanBlog: https://vanblog.mereith.com/
2. 使用 PicGo+腾讯云对象存储COS 作为图床 - 知乎: https://zhuanlan.zhihu.com/p/119250383

