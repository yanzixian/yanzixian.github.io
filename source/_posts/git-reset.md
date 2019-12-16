---
categories:
- git
---



> git-reset - Reset current HEAD to the specified state

主要有三个参数：`--soft`，`--mixed`，`--hard`
- `--soft`：仅移动当前HEAD指针，不会改变工作区和暂存区中的内容
- `--mixed`：默认参数，移动HEAD指针，改变暂存区内容，但不会改变工作区中的内容
- `--hard`：当前HEAD指针，暂存区和工作区中的内容均改变

具体可看[此处](https://www.jianshu.com/p/c2ec5f06cf1a)
