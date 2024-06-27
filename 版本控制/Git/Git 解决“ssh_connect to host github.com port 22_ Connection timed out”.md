# Git提交 “ssh:connect to host github.com port 22: Connection timed out”

## 产生原因

防火墙或者其他网络原因导致ssh连接方式 端口22被封锁

## 解决方案
### 方案一: 抛弃ssh连接方式，使用http连接

```bash
git config --local -e
```
将配置文件的url 从ssh连接方式,切换为http连接

![1676283125900.jpg](Git%20%E8%A7%A3%E5%86%B3%E2%80%9Cssh_connect%20to%20host%20github.com%20port%2022_%20Connection%20timed%20out%E2%80%9D.assets/eb912a312a081f3a13e7f74cd40e5959.1676283125900.jpg)

```bash
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = http://xxx.xxx.xxx.xxx:xxx/xxx/xxxx.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```
### 方案二: 替换22端口
```bash
# 1.进入~/.ssh下
cd ~/.ssh

# 2.创建一个config文件
vim config

# 3.编辑文件内容
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

Host gitlab.com
Hostname altssh.gitlab.com
User git
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

# 4.保存退出
# 5.检查是否成功
ssh -T git@github.com
```