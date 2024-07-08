# git常用指令

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
 # 获取远程分支到本地
 git checkout -b [branch-name] origin/[remote-branch-name]
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

