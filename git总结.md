# Git问题总汇

Author: ```haox```



## 2020/12/14 git rebase

```git rebase``` 变基,是通过让一个分支的修改合并到 **当前分支** 

举例: 我新建了**branch**名为```haox/fixbug``` ,但是目前 **主branch**:```master``` 分支并不是我新建分支时候的进度了,而是多了两个commit,这样我们

同时在提mr之前，确保你的改动都是基于最新的master，可以根据 git graph 的git图看出来。需要执行 rebase 的按照下面的操作进行（不懂的就git status，看git给到的提示进行操作）：

1、切换到master分支，git pull 最新的 master
2、切换回自己的分支，git rebase master
3、出现冲突进行解决【有冲突就截图看看一起解决一下】
4、git log 查看一下是否已经在当前自己的分支包含最新的master提交
5、在自己的分支上，git push -f 到 origin 远程自己的分支

```shell
git rebase 
while(存在冲突) {
    git status
    找到当前冲突文件，编辑解决冲突
    git add -u
    git rebase --continue
    if( git rebase --abort )
        break; //回到rebase之前
}
```



### 本地项目推送到远程git

> ``` git remote add origin yourGitUrl```
>
> 拉取一下`git pull origin master`



### 撤销修改（本地和远程）

- 本地
  - git reset 将commit拿出来提交到其他分支
    - git reset 目标版本号
    - git stash 放在暂存区
    - 切换到想提交的分支
    - git stash pop  释放暂存区代码
- 远程
  - git revert



### 挑选commit提交到新分支

git cherry-pick <分支哈希值（可以多个）>

### 遇到问题

- error: failed to push some refs to 'xxxx'
  - git pull --rebase origin yourbranch
  - git push origin yourbranch





