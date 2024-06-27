# HTTP userAgent 获取客户端访问途径:手机,pc,微信.....

附: 如果前端使用的是Layui框架,使用如下代码,即可获取到设备信息:

```css
	var device = layui.device();
	console.log(device);　　　　
　　　{

　　　　　os: "windows" //底层操作系统，windows、linux、mac等

　　　　　,ie: false //ie6-11的版本，如果不是ie浏览器，则为false

　　　　　,weixin: false //是否微信环境

　　　　　,android: false //是否安卓系统

　　　　　,ios: false //是否ios系统

　　　}
```
不想看下面解释直接拿去使用

```css
var os = function() {
     var ua = navigator.userAgent,
     isWindowsPhone = /(?:Windows Phone)/.test(ua),　　
     isSymbian = /(?:SymbianOS)/.test(ua) || isWindowsPhone, 　　//　塞班系统
     isAndroid = /(?:Android)/.test(ua), 　　//　安卓系统
     isFireFox = /(?:Firefox)/.test(ua), 　　//　火狐浏览器
     isChrome = /(?:Chrome|CriOS)/.test(ua),　　// google浏览器
     isTablet = /(?:iPad|PlayBook)/.test(ua) || (isAndroid && !/(?:Mobile)/.test(ua)) || (isFireFox && /(?:Tablet)/.test(ua)),　　//　平板
     isPhone = /(?:iPhone)/.test(ua) && !isTablet,　　// 苹果
     isPc = !isPhone && !isAndroid && !isSymbian;　　// 电脑PC
     return {
          isTablet: isTablet,
          isPhone: isPhone,
          isAndroid : isAndroid,
          isPc : isPc
     };
}();
　　// 使用
　　if(os.isAndroid) {
　　　　alert("安卓");
　　}else if(os.isPhone) {
　　　　alert("苹果");
　　}else if(os.isPC) {
　　　　alert("电脑");
　　}

```
可以根据自己需求,添加校验规则和返回属性
更多的校验规则可以看下方,没有的就需要自行百度

> js 不同浏览器的类型判断 navigator.userAgent - 南歌子 - 博客园       
> https://www.cnblogs.com/nangezi/p/10830257.html

获取客户端访问途径( userAgent)
(navigator.userAgent 是JS Browser对象中的一种)
JS  Browser 6大对象:
　　1.**window对象**(  浏览器中打开的窗口)
　　2.**Navigator对象**( 用户使用浏览器的信息)
　　3.**Screen对象**( 客户端显示屏幕的信息)
　　4.**History对象**( 包含用户（在浏览器窗口中）访问过的 URL)(可以通过window,history访问
　　5.**Location对象**( 包含有关当前 URL 的信息)
　　6.**存储对象**( sessionStorage （会话存储） 和 localStorage（本地存储）)
	**navigator.userAgent  是一种只读的字符串, 用于 HTTP 请求的用户代理头的值,所有的浏览器都支持,**

使用:

```css
var brower = {
     userAgent : function() {
           　　var ua = navigator.userAgent // 获取用户代理头
　　　　　　　　 var ualower = navigator.userAgent.toLocaleLowerCase();    // 按照本地方式把字符串转换为小写
              // 只有几种语言（如土耳其语）具有地方特有的大小写映射，所有该方法的返回值通常与 toLowerCase() 一样。
              return {
                    //根据代理头使用match()和index()方法,匹配对应浏览器的内核,将对应字段返回true或false;
            　　　　trident: ua.indexOf('Trident') > -1, // IE内核
            　　　　presto: ua.indexOf('Presto') > -1, // opera内核
              }
     }()   // ()不能忘
}
```
调用:

```css
if(brower. userAgent.trident) {
    // IE浏览器
}
else if(brower. userAgent.presto) {
       // opera浏览器
}
```
参考:

> (1条消息)js判断客户端是pc端还是移动端_kongjiea笔记-CSDN博客
> https://blog.csdn.net/kongjiea/article/details/17612899

> js 不同浏览器的类型判断 navigator.userAgent - 南歌子 - 博客园       
> https://www.cnblogs.com/nangezi/p/10830257.html