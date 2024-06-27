# 解决github主页图片不显示和访问慢的问题

> 参考自https://niter.cn/p/125

解决步骤：
①打开github后按**F12**，查看network，解析不出来的url都会有下滑红线标出

②访问[IPAddress.com](https://www.ipaddress.com/)，把报红线的url拷贝进去搜索IP地址。

③将地址写入本地host文件中，保存就行了。

本地host文件地址 `C:\Windows\System32\drivers\etc`下的hosts文件

```bash
140.82.112.5  api.github.com
192.30.253.112 github.com 
192.30.253.119 gist.github.com
151.101.184.133 assets-cdn.github.com
151.101.184.133 raw.githubusercontent.com
151.101.184.133 gist.githubusercontent.com
151.101.184.133 cloud.githubusercontent.com
151.101.184.133 camo.githubusercontent.com
151.101.184.133 avatars0.githubusercontent.com
151.101.184.133 avatars1.githubusercontent.com
151.101.184.133 avatars2.githubusercontent.com
151.101.184.133 avatars3.githubusercontent.com
151.101.184.133 avatars4.githubusercontent.com
151.101.184.133 avatars5.githubusercontent.com
151.101.184.133 avatars6.githubusercontent.com
151.101.184.133 avatars7.githubusercontent.com
151.101.184.133 avatars8.githubusercontent.com
```