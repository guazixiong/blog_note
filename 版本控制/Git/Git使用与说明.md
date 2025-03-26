## git常用指令

```bash
 #添加文件
 git add HEAD
 #添加所有改动文件
 git add .
 #提交命令
 git commit -m "提交日志"
 #推送
 git push
 #拉取
 git pull 
 #拉取分支
 git pull origin 分支名称
 
 
 # git命令撤销
 #撤销add
 git reset HEAD .
 #撤销commit消息内容
 git commit --amend
 #撤销commit,不撤销代码
 git reset --soft HEAD^
 #撤销commit,撤销代码
 git reset --hard HEAD^

 # 修改最近提交的 commit 信息
$ git commit --amend --message="modify message by daodaotest" --author="jiangliheng <jiang_liheng@163.com>"
# 仅修改 message 信息
$ git commit --amend --message="modify message by daodaotest"
# 仅修改 author 信息
$ git commit --amend --author="jiangliheng <jiang_liheng@163.com>"
 
 #从指定分支更新代码
 git pull origin 分支名称
 
 # 列出所有本地分支
 git branch
 
 # 列出所有远程分支
 git branch -r
 
 # 新建一个分支，但依然停留在当前分支
 git branch [branch-name]
 
 # 新建一个分支，并切换到该分支
 git checkout -b [branch]

 # 推送本地分支
 git push origin local-branch-name:local-branch-name
 
 # 合并指定分支到当前分支
 $ git merge [branch]
 
 # 删除分支
 $ git branch -d [branch-name]
 
 # 删除远程分支
 $ git push origin --delete [branch-name]
 $ git branch -dr [remote/branch]
```

## 什么是版本控制

版本控制（Revision control）是一种在开发的过程中用于管理我们对文件、目录或工程等内容的修改历史，方便查看更改历史记录，备份以便恢复以前的版本的软件工程技术。

- 实现跨区域多人协同开发(如svn)
- 追踪和记载一个或者多个文件的历史记录
- 组织和保护你的源代码和文档
- 统计工作量
- 并行开发、提高开发效率
- 跟踪记录整个软件的开发过程
- 减轻开发人员的负担，节省时间，同时降低人为错误

简单说就是用于管理多人协同开发项目的技术。

没有进行版本控制或者版本控制本身缺乏正确的流程管理，在软件开发过程中将会引入很多问题，如软件代码的一致性、软件内容的冗余、软件过程的事物性、软件开发过程中的并发性、软件源代码的安全性，以及软件的整合等问题。

无论是工作还是学习，或者是自己做笔记，都经历过这样一个阶段！我们就迫切需要一个版本控制工具！

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1664180306970-277ec4a9-903e-42c8-8be9-217777f3a402.webp)

多人开发就必须要使用版本控制！

## 常见的版本控制工具

主流的版本控制器有如下这些：

- **Git**
- SVN（Subversion）
- CVS（Concurrent Versions System）
- VSS（Micorosoft Visual SourceSafe）
- TFS（Team Foundation Server）
- **Visual Studio Online**

版本控制产品非常的多（Perforce、Rational ClearCase、RCS（GNU Revision Control System）、Serena Dimention、SVK、BitKeeper、Monotone、Bazaar、Mercurial、SourceGear Vault），现在影响力最大且使用最广泛的是Git与SVN

## 版本控制分类

**1、本地版本控制**

记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人用，如RCS。

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342693469-e3c66fd4-357a-4e4d-8a59-c68170f30522.webp)

**2、集中版本控制 SVN**

所有的版本数据都保存在服务器上，协同开发者从服务器上同步更新或上传自己的修改

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342693541-b1c86d1a-6f26-43ba-90b7-ac10235b3301.webp)

所有的版本数据都存在服务器上，用户的本地只有自己以前所同步的版本，如果不连网的话，用户就看不到历史版本，也无法切换版本验证问题，或在不同分支工作。而且，所有数据都保存在单一的服务器上，有很大的风险这个服务器会损坏，这样就会丢失所有的数据，当然可以定期备份。代表产品：SVN、CVS、VSS

**3、分布式版本控制****Git**

**每个人都拥有全部的代码！安全隐患！**

所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。

不会因为服务器损坏或者网络问题，造成不能工作的情况！

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342693526-939596b7-a823-455e-acc8-11db501cbfde.webp)

Git与SVN的主要区别

**SVN是集中式版本控制系统，版本库是集中放在中央服务器的**，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。**集中式版本控制系统是必须联网才能工作，对网络带宽要求较高**。

**Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了**，因为版本都在自己电脑上。协同的方法是这样的：比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件！

**Git是目前世界上最先进的分布式版本控制系统。**

## Git历史

Git是免费、开源的，最初Git是为辅助 Linux 内核开发的，来替代 BitKeeper！

视频同步笔记：狂神聊Githttps://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ

## Git安装与环境配置

视频同步笔记：狂神聊Githttps://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ

所有的配置文件，其实都保存在本地！

```bash
#查看配置 
 git config -l

#查看系统config
 git config --system --list
 　　
#查看当前用户（global）配置
 git config --global  --list
```

**Git相关的配置文件：**

1）Git\etc\gitconfig ：Git 安装目录下的 gitconfig --system 系统级

2）C:\Users\Administrator\ .gitconfig 只适用于当前登录用户的配置 --global 全局

这里可以直接编辑配置文件，通过命令设置后会响应到这里。

设置用户名与邮箱（用户标识，必要）

当你安装Git后首先要做的事情是设置你的用户名称和e-mail地址。这是非常重要的，因为每次Git提交都会使用该信息。它被永远的嵌入到了你的提交中：

```bash
git config --global user.name "kuangshen"  #名称  
git config --global user.email "24736743@qq.com"   #邮箱 
```

只需要做一次这个设置，如果你传递了--global 选项，因为Git将总是会使用该信息来处理你在系统中所做的一切操作。

如果你希望在一个特定的项目中使用不同的名称或e-mail地址，你可以在该项目中运行该命令而不要--global选项。总之--global为全局配置，不加为某个项目的特定配置。

## Git基本理论（重要）

三个区域

Git本地有三个工作区域：**工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository或Git Directory)**。如果在加上远程的**git仓库**(Remote Directory)就可以分为**四个工作区域**。文件在这四个区域之间的转换关系如下：

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342693509-c1210167-d1a1-40fa-8343-910a2c48fd92.webp)

- **Workspace：工作区，就是你平时存放项目代码的地方**
- **Index / Stage：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息**
- **Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本**
- **Remote：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换**

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342693510-7f6b8983-b9fc-414f-ab20-e0ccd74b64dd.webp)

- **Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。**
- **WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。**
- **.git：存放Git管理信息的目录，初始化仓库的时候自动创建。**
- **Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。**
- **Local Repo：本地仓库，一个存放在本地的版本库；HEAD会只是当前的开发分支（branch）。**
- **Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。**

工作流程

git的工作流程一般是这样的：

１、在工作目录中添加、修改文件；

２、将需要进行版本管理的文件放入暂存区域；

３、将暂存区域的文件提交到git仓库。

因此，git管理的文件有三种状态：**已修改（modified）,已暂存（staged）,已提交(committed)**

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342694020-ad8b3839-4f06-48b4-92b1-e196f6431389.webp)

## git项目搭建

工作目录（WorkSpace)一般就是你希望Git帮助你管理的文件夹，可以是你项目的目录，也可以是一个空目录，建议不要有中文。

### 本地仓库搭建

创建本地仓库的方法有两种：一种是创建全新的仓库，另一种是克隆远程仓库。

1、创建全新的仓库，需要用GIT管理的项目的根目录执行：

```bash
 # 在当前目录新建一个Git代码库
 $ git init
```

2、执行后可以看到，仅仅在项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面。

### 克隆远程仓库

1、另一种方式是克隆远程目录，由于是将远程服务器上的仓库完全镜像一份至本地！

```bash
 # 克隆一个项目和它的整个代码历史(版本信息)
 $ git clone [url]  
# git clone https://gitee.com/kuangstudy/openclass.git
```

## Git文件操作

文件的四种状态

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。

- **Untracked: 未跟踪**, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过**git add 状态变为Staged**.[新增文件]
- **Unmodify**: 文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为Modified. 如果使用**git rm移出版本库, 则成为Untracked文件**[原有文件]
- **Modified: 文件已修改, 仅仅是修改**, 并没有进行其他的操作. 这个文件也有两个去处, 通过**git add可进入暂存staged状态**, 使用**git checkout 则丢弃修改过, 返回到unmodify状态**, 这个git checkout即从库中取出文件, 覆盖当前修改
- Staged: 暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态. 执行git reset HEAD filename取消暂存, 文件状态为Modified[待提交文件]

```bash
 #查看文件状态
 #查看指定文件状态
 git status [filename]
 
 #查看所有文件状态
 git status
 
 # git add .                  添加所有文件到暂存区
 # git commit -m "消息内容"    提交暂存区中的内容到本地仓库 -m 提交信息
```

忽略文件

有些时候我们不想把某些文件纳入版本控制中，比如数据库文件，临时文件，设计文件等

在主目录下建立".gitignore"文件，此文件有如下规则：

1. **忽略文件中的空行或以井号（#）开始的行将会被忽略。**
2. **可以使用Linux通配符。例如：星号（\*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选的字符串等。**
3. **如果名称的最前面有一个感叹号（!），表示例外规则，将不被忽略。**
4. **如果名称的最前面是一个路径分隔符（/），表示要忽略的文件在此目录下，而子目录中的文件不忽略。**
5. **如果名称的最后面是一个路径分隔符（/），表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。**

```bash
 #为注释
 *.txt        #忽略所有 .txt结尾的文件,这样的话上传就不会被选中！
 !lib.txt     #但lib.txt除外
 /temp        #仅忽略项目根目录下的TODO文件,不包括其它目录temp
 build/       #忽略build/目录下的所有文件
 doc/*.txt    #会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

## 使用码云

github 是有墙的，比较慢，在国内的话，我们一般使用 gitee ，公司中有时候会搭建自己的gitlab服务器

1、注册登录码云，完善个人信息

2、设置本机绑定SSH公钥，实现免密码登录！（免密码登录，这一步挺重要的，码云是远程仓库，我们是平时工作在本地仓库！)

3、将公钥信息public key 添加到码云账户中即可！

4、使用码云创建一个自己的仓库！

许可证：开源是否可以随意转载，开源但是不能商业使用，不能转载，... 限制！

## IDEA中集成Git

1、新建项目，绑定git。

将我们远程的git文件目录直接copy到项目即可

注意观察idea中的变化

2、修改文件，使用IDEA操作git。

- **添加到暂存区**

```bash
git add 文件
git add .
```

- **commit 提交**

```bash
 git commit -m "提交日志" 
```

- **push到远程仓库**

```bash
 git push 
```

3、提交测试

## GIT分支

分支在GIT中相对较难，分支就是科幻电影里面的平行宇宙，如果两个平行宇宙互不干扰，那对现在的你也没啥影响。不过，在某个时间点，两个平行宇宙合并了，我们就需要处理一些问题了！

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342694043-1cf95024-7710-497d-9909-fbc652f14d4f.webp)

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342694241-98cd9f30-5976-439d-86a8-c0a5086fbf8a.png)

### 创建与合并分支 - 廖雪峰的官方网站

https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424

直接把master指向dev的当前提交，就完成了合并

### 分支冲突

**当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。**

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

在合并出现问题之后，master分支的装填是merging,代表正在合并过程中

解决冲突 - 廖雪峰的官方网站https://www.liaoxuefeng.com/wiki/896043488029600/900004111093344

### Bug分支

```bash
 git stash   #存储现场
 git stash list #查看现场
 git stash apply #恢复现场
 git stash drop #删除现场
 git stash pop   #恢复的同时删除现场
 git stash apply stash@{0} #恢复指定现场
```

\#主分支出现bug修复后,其他分支也需要修复这个bug git cherry-pick 4c805e2 #复制一个特定的提交到当前分支

Bug分支 - 廖雪峰的官方网站https://www.liaoxuefeng.com/wiki/896043488029600/900388704535136

### IDEA中操作git分支

![img](Git%E4%BD%BF%E7%94%A8%E4%B8%8E%E8%AF%B4%E6%98%8E.assets/1645342694518-7345abdd-da3c-46ff-a924-b8b23597d81a.png)

如果同一个文件在合并分支时都被修改了则会引起冲突：解决的办法是我们可以修改冲突文件后重新提交！选择要保留他的代码还是你的代码！

**master主分支应该非常稳定，用来发布新版本，一般情况下不允许在上面工作**，工作一般情况下在**新建的dev分支**上工作，工作完后，比如上要发布，或者说**dev分支代码稳定后可以合并到主分支master上来**。

## ssh公钥

### 查看SSH公钥

```bash
cd ~/.ssh
cat id_rsa.pub
 
# 或
 cat ~/.ssh/id_rsa.pub
# 或windows下ssh存放目录 C:\Users\lenovo.ssh\id_rsa.pub
```

### 生成SSH公钥

#### 配置git信息

```bash
  git config --global user.name "用户名"
  git config --global user.email "邮箱"
```

#### 生成ssh

```bash
 ssh-keygen -t rsa -C "邮箱" 
```

#### 查看ssh

```bash
  cd ~/.ssh   cat id_rsa.pub     
#或者   
	cat ~/.ssh/id_rsa.pub 
```

## git 版本回退

切记，回退分支时一定要备份！备份！备份！

### 查看提交历史

```bash
 git log
 commit c8fd53763bbe8b000cea0fbcacf7fe1a9ffd57ff (HEAD -> master, origin/master, origin/HEAD)
 Date:   Sat Jan 1 22:49:03 2022 +0800
 
     新增文档
     
  .....
```

格式解析：

commit 版本号（HEAD -> master, origin/master, origin/HEAD 代表当前指向）

提交人

提交时间

除了通过命令行可以查看版本号,也可以在github或者gitee下 ,提交历史中查看版本号

### 使用“git reset --hard 目标版本号”命令将版本回退

```bash
 git reset --hard c8fd53763bbe8b000cea0fbcacf7fe1a9ffd57ff
```

### 使用“git push -f ” 提交更改

此处不使用git push,因为我们刚刚修改了本地的HEAD指向,导致比远程库的版本要旧,所以会报错

使用git push -f 强制将本地版本推上去

```bash
 git push -f 
```

## git 删除本地和远程分支

```bash
# 查看所有分支
git branch -a

# 删除本地分支
git branch -d Chapater8
 
# 删除远程分支
git push origin --delete Chapater6
```

## Git 远程分支及提交时间排序

以下是按提交时间从小到大排序的远程分支及其最后一次提交时间：

```bash
git for-each-ref --format '%(committerdate:iso8601) %(refname:short)' refs/remotes/ | sort
```

示例输出：

```bash
2025-03-23T08:30:12+08:00 origin/feature-xyz
2025-03-24T10:30:12+08:00 origin/main
2025-03-25T14:23:47+08:00 origin/another-branch  
...
```

如果您希望按时间倒序排序，可以使用以下命令：

```bash
git for-each-ref --format '%(committerdate:iso8601) %(refname:short)' refs/remotes/ | sort -r
```

示例输出（倒序）：

```bash
2025-03-25T14:23:47+08:00 origin/another-branch
2025-03-24T10:30:12+08:00 origin/main
2025-03-23T08:30:12+08:00 origin/feature-xyz
...
```

## github主页图片不显示和访问慢的问题

### 解决步骤：

1. **打开github后按F12**，查看network，解析不出来的url都会有下滑红线标出
2. **访问**[IPAddress.com](https://www.ipaddress.com/)**，把报红线的url拷贝进去搜索IP地址。**
3. **将地址写入本地host文件中(C:\Windows\System32\drivers\etc**)

## git 如何撤销 commit、git commit 提交之后如何取消本次提交、如何更改提交的内容

可以先用 git reflog 查看历史提交记录

## 软撤销 --soft

本地代码不会变化，只是 git 转改会恢复为 commit 之前的状态

不删除工作空间改动代码，撤销 commit，不撤销 git add .

```bash
git reset --soft HEAD~1
# 或
git reset --soft HEAD^
```

表示撤销最后一次的 commit ，1 可以换成其他更早的数字

是~,不是-

## 硬撤销

本地代码会直接变更为指定的提交版本，慎用

删除工作空间改动代码，撤销 commit，撤销 git add .

注意完成这个操作后，就恢复到了上一次的commit状态。

```bash
git reset --hard HEAD~1
```

git reset --hard HEAD~1 

如果仅仅是 commit 的消息内容填错了

```bash
git commit --amend
```

进入 vim 模式，对 message 进行更改

还有一个 --mixed

```bash
git reset --mixed HEAD~1
```

意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作 这个为默认参数,git reset --mixed HEAD~1 和git reset HEAD~1效果是一样的。