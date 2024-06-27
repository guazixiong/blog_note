```bash
# 打开命令窗口(管理员模式启动,否则提示无权)
windows + R 

# 查找所有运行的端口
netstat -ano

活动连接

  协议  本地地址          外部地址        状态           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       536
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:4523           0.0.0.0:0              LISTENING       26488
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       6852
  TCP    0.0.0.0:6000           0.0.0.0:0              LISTENING       24696
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       11300
 	...		 ...                    ...                    ...             ...
  ...		 ...                    ...                    ...             ...
  ...		 ...                    ...                    ...             ...

# 查看被占用端口对应的 PID,PID 为11300
netstat -ano | findstr "8081"

  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       11300

# 查看指定 PID 的进程
tasklist | findstr "11300"

node.exe                  11300 Services                   0     20,128 K

# 结束进程 强制（/F参数）杀死 pid 为 9088 的所有进程包括子进程（/T参数）：
taskkill /T /F /PID 9088 

```

