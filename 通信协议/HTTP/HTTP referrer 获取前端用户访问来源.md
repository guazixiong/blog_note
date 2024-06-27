# HTTP referrer 获取前端用户访问来源

## 一、介绍

HTTP Header referer 提供访问当前页面来源的信息
很有趣的是，这个字段的拼写是错的。
**Referer**的正确拼写是**Referrer**，但是写入标准的时候，不知为何，没人发现少了一个字母r。
标准定案以后，只能将错就错，所有头信息的该字段都一律错误拼写成Referer。(**但通过js引用时是 document.referrer**)

```css
console.log(document.referer);  // JS获取http referer

$_SERVER['HTTP_REFERER'];　　	// php获取http referer
```

## 二、应用场景

Referer字段的逻辑是这样的，用户在地址栏输入网址，或者选中浏览器书签，就不发送Referer字段。

**不发送Referer字段场景:**

 1. 直接输入地址
 2. 使用location.reload()刷新（location.href或者location.replace()刷新有信息）；
 3. QQ,微信点击链接进入自身的浏览器 
 4. 从https的网站直接进入一个http协议的网站
 5. a标签在rel设置noreferer


```css
rel="noreferrer"
```

 6. 通过设定referer policy 值,在不同条件下控制referer的发送

**以下三种场景，会发送Referer字段。**

 1. 用户点击网页上的链接。 
 2. 用户发送表单。
 3. 网页加载静态资源，比如加载图片、脚本、样式。

三、使用
Referer字段实际上告诉了服务器，用户在访问当前资源之前的位置。这往往可以用来用户跟踪。(**如无需登录即可使用功能界面**)

**安全性:**
由于涉及隐私，很多时候不适合发送Referer字段。

```css
<a href="..." rel="noreferrer" target="_blank">xxx</a> 
//referrer正确拼写
```
Referer并不是安全的,因为可以通过**插件修改发送头信息中的referer**
**解决:**

>  Referer字段包含源信息、路径和查询字符串，不包含锚点、用户名和密码。  
>  注意区分Referer字段和源信息(协议+域名+端口)

 W3C  Referrer Policy 设定8个值,在特定情况下发送,


 1. no-referrer   不发送
 2. no-referrer-when-downgrade   HTTPS->HTTP(浏览器默认行为),不发送
 3. same-origin   同源(协议+域名+端口)不发送
 4. origin       不管是否跨域,Referer字段都只发送源信息(协议+域名+端口)
 5. strict-origin   HTTPS->HTTP 不发送referer字段,其他只发送源信息
 6. origin-when-cross-origin   同源,发送referer完整信息,包括跨域
 7. strict-origin-when-cross-origin    同源时，发送完整的Referer字段；跨域时，如果 HTTPS 网址链接到 HTTP 网址，不发送Referer字段，否则发送源信息。
 8. unsafe-url   Referer字段包含源信息、路径和查询字符串，不包含锚点、用户名和密码。

**页面重定向**
定制**中间界面**,通过A->B(中间)->C  C获取到的是中间界面B,可以有效保护A界面数据不被泄露.

参考:

> 如何在前端统计用户访问来源_weixin_30484247的博客-CSDN博客
> https://blog.csdn.net/weixin_30484247/article/details/98705127?utm_medium=distribute.pc_relevant.none-task-blog-title-1&spm=1001.2101.3001.4242

> HTTP Referer 教程 - 阮一峰的网络日志
> http://www.ruanyifeng.com/blog/2019/06/http-referer.html
>

> JS获取上一访问页面URL地址document.referrer实践 « 张鑫旭-鑫空间-鑫生活
> https://www.zhangxinxu.com/wordpress/2017/02/js-page-url-document-referrer/