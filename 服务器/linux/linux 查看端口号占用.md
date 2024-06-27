# 查看Linux上特定端口被那个进程占用

使用以下命令来查找特定端口号（例如，端口号80）的进程ID（PID）：

```bash
[root@bigdata212 pos]# lsof -i:8080  # 查看8080端口占用程序
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java     2213 root  215u  IPv6 139083      0t0  TCP big:38886->big:8080 (CLOSE_WAIT)
java     2213 root  244u  IPv6 134016      0t0  TCP big:38892->big:8080 (CLOSE_WAIT)
java     2213 root  254u  IPv6 107916      0t0  TCP big:39152->big:8080 (CLOSE_WAIT)
java     2213 root  267u  IPv6 132607      0t0  TCP big:38882->big:8080 (CLOSE_WAIT)
java     2213 root  268u  IPv6 132610      0t0  TCP big:38904->big:8080 (CLOSE_WAIT)
java     2213 root  273u  IPv6 129444      0t0  TCP big:38916->big:8080 (CLOSE_WAIT)
java     2213 root  276u  IPv6  57235      0t0  TCP big:39160->big:8080 (CLOSE_WAIT)
java     2213 root  280u  IPv6 132787      0t0  TCP big:39164->big:8080 (CLOSE_WAIT)
java     2213 root  281u  IPv6 144296      0t0  TCP big:39206->big:8080 (CLOSE_WAIT)
java     2213 root  282u  IPv6 142689      0t0  TCP big:39182->big:8080 (CLOSE_WAIT)
java     2213 root  284u  IPv6 107931      0t0  TCP big:39192->big:8080 (CLOSE_WAIT)
java     2213 root  285u  IPv6  59364      0t0  TCP big:39470->big:8080 (CLOSE_WAIT)
java    11460 root  220u  IPv6  56456      0t0  TCP *:30096 (LISTEN)
```
查看端口占用程序所在位置
```bash
[root@big pos]# sudo ls -l /proc/2213/cwd  # 查看进程PID 2213程序所在位置
lrwxrwxrwx 1 root root 0 Oct 23 10:34 /proc/2213/cwd -> /home/use/gateway-0.0.2
[root@big pos]# sudo ls -l /proc/11460/cwd # 查看进程PID 11460程序所在位置
lrwxrwxrwx 1 root root 0 Oct 23 10:35 /proc/11460/cwd -> /home/erp/posmanage-service0.02
```
kill掉程序
```bash
kill 11460 # 终止进程
kill -9 11460 # 强制终止进程，可能会导致数据丢失或进程不正常退出
```
