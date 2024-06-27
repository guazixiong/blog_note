# QRCode 生成和下载二维码

> 使用注意事项:
> 1.url必须带通信协议
> 2.生成二维码前必须先清空二维码,不然会重复生成二维码（文章最后）


1、引入插件QRCode
**QRCode.js 是一个用于生成二维码的 JavaScript 库**。主要是通过获取 DOM 的标签,再通过 **HTML5 Canvas 绘制**而成,**不依赖任何库**。

> (偷懒链接) 
> https://static.runoob.com/download/qrcodejs-04f46c6.zip

2、基本使用

```css
<div id="qrcode"></div>
var qrcode = new QRCode(document.getElementById("qrcode"), "http://www.runoob.com"); // 设置要生成二维码的链接
qrcode.makeCode();     // 生成二维码
......
qrcode.clear();        //清除二维码
```
3、QRCode 生成和下载二维码 完整代码

```css
<html>
	<head>
		<meta charset="utf-8">
		<title>生成和下载二维码</title>
		<!-- 引入QRCode -->
		<script src="qrcodejs-04f46c6/qrcode.min.js"></script>
		<script src="qrcodejs-04f46c6/jquery.min.js"></script>
	</head>
	<body>
		<button id="p1">点击生成二维码</button>
		<button id="p2">下载二维码</button>
		<a id="downloadLink" style="display: none;"></a>
		<!-- 显示二维码区域 -->
		<div id="qrcode" style="width:100px; height:100px; margin-top:15px;"></div>
		<script type="application/javascript">
			$("#p1").click(function() {
				// 避免因多次点击生成多个二维码
				document.getElementById("qrcode").innerHTML = "";
				// 二维码扫描后跳转界面
				var url = "http://www.baidu.com";	// 必须添加访问url的http协议,否则显示字符串
				// 写法一
				var qrcode = new QRCode(document.getElementById("qrcode"), {
					// 二维码基本参数
					text: url,		//	url
					width: 128,			// 宽度(百分比 no)
					height: 128,		// 高度(百分比 no)
					colorDark : "#000000",	// 生成的二维码的深色部分
					colorLight : "#ffffff",	// 生成二维码的浅色部分
					correctLevel : QRCode.CorrectLevel.H	// 容错级别
					background : "#ffffff",//背景颜色  
					foreground : "#000000",//前景颜色
					src : '../img/pm.jpg'  //二维码中间的图片
				});
				// 生成二维码
				qrcode.makeCode(url);
				
				
				/* // 写法二
				var qrcode = new QRCode(document.getElementById("qrcode"), {
					text: "http://www.baidu.com",		//	url
					width: 128,			// 宽度(百分比 no)
					height: 128,		// 高度(百分比 no)
					colorDark : "#000000",	// 生成的二维码的深色部分
					colorLight : "#ffffff",	// 生成二维码的浅色部分
					correctLevel : QRCode.CorrectLevel.H	// 容错级别
					background : "#ffffff",//背景颜色  
					foreground : "#000000",//前景颜色
					src : '../img/pm.jpg'  //二维码中间的图片
				});
				qrcode.makeCode(); 
			});*/
			
			$("#p2").click(function() {
				var name= "QRcode";
				// 获取base64的图片节点
				var img = document.getElementById('qrcode').getElementsByTagName('img')[0];	
				// 构建画布
				var canvas = document.createElement('canvas');
				canvas.width = img.width;
				canvas.height = img.height;
				canvas.getContext('2d').drawImage(img, 0, 0);
				// 构造url
				url = canvas.toDataURL('image/png');
				// 构造a标签并模拟点击
				var downloadLink = document.getElementById('downloadLink');
				downloadLink.setAttribute('href', url);
				downloadLink.setAttribute('download', name+'.png');
				downloadLink.click();
			  });
		</script>
	</body>
</html>
```
附：解决重复生成二维码
（但感觉QRCode附带的qrcode.clear()挺鸡肋的,不如直接使用js置空）

```css
// 每次添加二维码时先清空(上述例子已应用)
document.getElementById("qrcode").innerHTML = "";
```

```css

//QRCode自带的clear()清除二维码,并重新生成二维码
var code;
$("#p1").click(function() {
	if(code !=null) {
		code.clear();
	}
	var url = "http://www.baidu.com";	
	var qrcode = new QRCode(document.getElementById("qrcode"), {
		// 二维码基本参数
		text: "http://www.baidu.com",		//	url
		......
	});
	// 生成二维码
	qrcode.makeCode(url);
	code = qrcode;
});
```