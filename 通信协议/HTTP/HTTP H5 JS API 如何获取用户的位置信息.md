# HTTP H5 JS API 如何获取用户的位置信息?

+ H5 Geolocation 获取用户位置的经纬度
+ 获取用户位置的具体信息,调用API
+ 只想简单获取用户所在省市,建议使用js

## 一、H5自带方法 Geolocation (获取的是经纬度，需要自己引入第三方进行查询)

**Geolocation（地理定位）**对于拥有 GPS 的设备，比如 iPhone，地理定位更加精确。

> 属性详情：
> HTML5 地理定位 | 菜鸟教程      (H5  Geolocation)
> https://www.runoob.com/html/html5-geolocation.html


## 二、通过api接口,百度地图,腾讯地图,需要申请key

 1. 申请api接口的key
 2. 引入对应api接口的js文件(更改为自己的密钥)
 3. 根据官方文档编写代码获取用户访问位置
下面以百度地图作为例子获取用户位置信息

```css
<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=7a6QKaIilZftIMmKGAFLG7QT1GLfIncg"></script>
// 密钥建议自己申请
<script>
	var geolocation = new BMap.Geolocation(); 
    var gc = new BMap.Geocoder();     
    geolocation.getCurrentPosition(function(r) {  
        if (this.getStatus() == BMAP_STATUS_SUCCESS) { 
        	var a = r.point.lng;
        	var b = r.point.lat ; 
        	var pt = r.point; 
            gc.getLocation(pt, function(rs) {   
	            var addComp = rs.addressComponents;
		    	var province = res.province;	// 省
		   	 	var city = res.city;	// 市
		    	var county = res.county;	// 区
		    	var street = addComp.street;	// 街道
		    	var streetNumber = addComp.streetNumber;	// 门牌号
	       		alert("成功获取当前位置，定位可能出现偏差！");
        	}); 
	}else{  
            switch( this.getStatus() ){  
                    case 2:  
                        alert( '位置结果未知 获取位置失败.' );  
                        break;  
                    case 3:  
                        alert( '导航结果未知 获取位置失败..' );  
                        break;  
                    case 4:  
                        alert( '非法密钥 获取位置失败.' );  
                        break;  
                    case 5:  
                        alert( '对不起,非法请求位置  获取位置失败.' );  
                        break;  
                    case 6:  
                        alert( '对不起,当前 没有权限 获取位置失败.' );  
                        break;  
                    case 7:  
                        alert( '对不起,服务不可用 获取位置失败.' );  
                        break;  
                    case 8:  
                        alert( '对不起,请求超时 获取位置失败.' );  
                        break;
            }
        }          
    },{enableHighAccuracy: true});
<script>      
```

## 3.直接通过js去获取用户位置信息
目前找到两种方式可以有效获取用户位置信息;

 1. 新浪IP地址查询接口,需要手动传递ip（）

```css
访问 http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=js&ip=219.242.98.111
```

 2. 搜狐IP地址查询接口 (简单粗暴)

```css
访问 http://pv.sohu.com/cityjson?ie=utf-8
```

```css
<script src="https://pv.sohu.com/cityjson?ie=utf-8" type="text/javascript"></script>
<script>
      console.log(returnCitySN);
</script>
```

```css
returnCitySN = {"cip": "127.0.0.1", "cid": "35000", "cname": "xxx省xxx市"};
```