# /usr/local/bin/docker-compose:行1: html: 没有那个文件或目录

执行`docker-compose version`提示报错

![1700577352508](usrlocalbindocker-compose%E8%A1%8C1%20html%20%E6%B2%A1%E6%9C%89%E9%82%A3%E4%B8%AA%E6%96%87%E4%BB%B6%E6%88%96%E7%9B%AE%E5%BD%95.assets/1700577352508.webp)

解决:

直接在release中下载对应的linux发行版

Release v2.18.1 · docker/compose: https://github.com/docker/compose/releases/tag/v2.18.1

下载完后将软件上传至 Linux的【/usr/local/bin】目录下

```bash
# 重命名
[root@VM-8-7-centos bin]# sudo mv docker-compose-linux-x86_64 docker-compose
# 将可执行权限应用于二进制文件
[root@VM-8-7-centos bin]# sudo chmod +x /usr/local/bin/docker-compose
# 创建软连接
[root@VM-8-7-centos bin]# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 检验是否安装完成
[root@VM-8-7-centos bin]# docker-compose -v
Docker Compose version v2.18.1
```

