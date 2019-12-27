---
title: git push git push 时出现non-fast- forward的解决方法
categories: git
tags:
	- git push
---

#### git push 时出现`non-fast- forward`的解决方法

出现原因：出现此错误原因大概率是远程仓库有其他人提交新的`commit`，导致本地仓库没有远程仓库的提交历史记录

###### 解决方法：

1. ```git
   git fetch origin remote_branch //拉取远程仓库分支更新
   git merge origin remote_branch //合并更新
   git pull origin remote_branch  //更新本地分支
   ```

   再执行`push`操作

2. ```git
   git pull origin remote_branch --allow-unrelated-histories //允许合并无关的提交历史
   ```

   再执行`push`操作