# Linux 后台运行jar,并指定文件输出

在linux服务器上运行Jar文件时通常的方法是：

```bash
# ssh窗口关闭时，程序中止运行.或者是运行时没法切出去执行其他任务
$ java -jar test.jar
```

## nohup

```bash
# nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行
# 当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中,除非另外指定了输出文件。

# 默认输出到nohup.out文件中
$ nohup java -jar test.jar &

# 指定输出到mylog中
$ nohup java -jar test.jar >mylog &
```

## jobs 和 fg

```bash
$ jobs
# 那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。
# 如果想将某个作业调回前台控制，只需要 fg + 编号即可。
$ fg 2
```

## 查看某端口占用的线程的pid

```bash
# 8080端口是否被占用 
netstat -ano |grep 8080
# kill
kill 257982
```