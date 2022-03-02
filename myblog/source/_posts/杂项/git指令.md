### 暂时将未提交的变化移除，稍后再移入

```
git stash
```

```
git stash pop
```

合并远程分支到本地

```
git merge origin/dev
```
【Git】撤回已经commit到本地的提交记录

git bash直接干到你的code.
直接敲命令: git reset --soft HEAD~1
搞定
就是这么简单粗暴. 如有顾虑请自行找个案例测试即可.

### 需求：回滚到某个提交记录

git reset --hard 提交记录ID

### 需求：需要将A分支的某次提交记录 ，合并到B分支

解决步骤：

1）git checkout A分支

　　找到提交的commit id 可以使用git log 命令 或者 右键上次提交的记录 copy reversion number

2) 切回到 B分支 

　　使用 git cherry-pick 提交记录ID ，回车即可

 

如果遇到问题，可以使用 git cherry-pick --abort 命令 放弃本次合并的需要
