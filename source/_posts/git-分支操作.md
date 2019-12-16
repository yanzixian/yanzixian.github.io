---
categories:
- git
---



####基本命令：

- 创建分支：`git branch <name>`
- 切换分支：`git checkout <name>`
- 创建并切换分支：`git checkout -b <name>`
- 查看本地分支：`git branch`
- 查看远程分支：`git branch -r`
- 查看所有分支：`git branch -a`
- 删除本地分支：`git branch -d <name>`，分支存在未merge的提交，会删除失败，可以使用`git branch -D <name>`强制删除
- 删除远程分支： `git push origin --delete <remote_name>`，也可以使用`git push origin :<remote_name>`，省略本地分支名，push一个空分支，即删除对应的远程分支
- 查看本地分支与远程分支的对应：`git branch -vv`
- 合并某分支到当前分支：`git merge <name>`
####本地分支与远程分支的关联：
建立关联的方式：
- 手动建立关联：
  - 本地分支与远程分支均已存在时：`git branch --set-upstream-to=origin/<remote_name> <local_name>`，`origin`是远程主机名，通常通过`git add`命令添加
  - 已有本地分支，需要建立对应的远程分支：`git push --set-upstream origin <local_name>`，新建的远程分支与本地分支同名
  - 已有远程分支，需要新建对应的本地分支：`git checkout --track origin/<remote_name>`，新建的本地分支与远程分支同名
- push时建立关联：`git push -u origin <local_name>`，通过`-u`参数，将本地分支与远程同名分支建立关联，若不同名，则需指定分支名，`git push -u origin <local_name> <remote_name>`，第一次使用`-u`建立关联后，后续push操作可省略`-u`参数
- 新建分支时建立关联：`git checkout -b <local_name> origin/<remote_name>`


