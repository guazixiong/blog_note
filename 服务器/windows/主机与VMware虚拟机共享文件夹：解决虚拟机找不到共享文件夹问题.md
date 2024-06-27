# 主机与VMware虚拟机共享文件夹：解决虚拟机找不到共享文件夹问题

> 原文链接: 主机与VMware虚拟机共享文件夹：解决虚拟机找不到共享文件夹问题 - 知乎: https://zhuanlan.zhihu.com/p/650638983

主机与虚拟机之间传递文件，最快捷的方法莫过于共享文件夹。此方法不需要复制文件，而且可以节省硬盘空间。

具体设置步骤如下：

打开 “虚拟机 -> 编辑虚拟机设置 -> 选项 -> 共享文件夹”

![img](%E4%B8%BB%E6%9C%BA%E4%B8%8EVMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%9A%E8%A7%A3%E5%86%B3%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E9%97%AE%E9%A2%98.assets/1713798590442-fb918af4-fde0-4b82-94b7-75c64e999498.png)

将`共享文件夹`设置为`总是启用`,添加自定义共享文件夹目录,我这里添加`E:\centos-7-share`作为共享目录

![img](%E4%B8%BB%E6%9C%BA%E4%B8%8EVMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%9A%E8%A7%A3%E5%86%B3%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E9%97%AE%E9%A2%98.assets/1713798718178-a35300f3-7fb2-4bcc-a90a-86bf3cb199ab.png)

![img](%E4%B8%BB%E6%9C%BA%E4%B8%8EVMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%9A%E8%A7%A3%E5%86%B3%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E9%97%AE%E9%A2%98.assets/1713798712133-b0a9eb37-cbbd-499a-98d5-7bd476f620f5.png)

![img](%E4%B8%BB%E6%9C%BA%E4%B8%8EVMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%9A%E8%A7%A3%E5%86%B3%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E9%97%AE%E9%A2%98.assets/1713798725196-0a37f279-73c6-41df-a1fb-86ed0d161b85.png)

网上很多教程到这一步就结束了。

然而，自己在虚拟机中并未找到共享的两个文件夹。

因为我们还缺少一个重要步骤：挂载操作

具体命令如下：

```bash
$ sudo mount -t fuse.vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

`/mnt/hgfs/` 是挂载点，我们也可以修改为其它挂载点
`-o allow_other` 表示普通用户也能访问共享目录。

然后，**再次进入 /mnt/hgfs** 查看 （注意：挂载后必须要再次进入/mnt/hgfs才能查看到共享的文件夹）

```bash
$ cd /mnt/hgfs
$ ls
```

![img](%E4%B8%BB%E6%9C%BA%E4%B8%8EVMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%9A%E8%A7%A3%E5%86%B3%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%BE%E4%B8%8D%E5%88%B0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E9%97%AE%E9%A2%98.assets/1713798917497-feb04871-3529-43dd-b375-14406cecb830.png)

**注意：如果虚拟机重启，需要再次挂载共享文件夹。**