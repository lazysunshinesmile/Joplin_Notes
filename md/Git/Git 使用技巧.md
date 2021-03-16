Git 使用技巧

Git 使用技巧

[TOC]
## 恢复单个文件：
`git checkout file`
## 同时恢复多个被删除的文件：
`git ls-files -d | xargs -i git checkout {}`
## 将某个修改追加到某次提交

1. 如果是追加到上一次commit
`git commit --amend`
2. 前几次的某次提交
    - `git stash` 保存工作空间的改动
    - `git rebase <指定commit的父commit> --interactive`
    - 将需要改动的commit前面的pick 改为 edit,然后保存退出
    - `git stash pop`
    - `git add <更改的文件> `
    - `git commit --amend`
    - `git rebase --continue`
    - 
## 修改某一次提交
1. `git log . ` 有三次提交
```
first：
secend:
last:
```
2. 欲修改secend 为second，`git rebase -i last_commit_id`
```
pick firt...
pick secend...
```
3. 修改pick为r，然后:wq
```
pick firt...
r secend...
```
4. 跳出secend的提交信息
5. 修改保存退出

## git查看stash中的详细改变
`git stash show -p stash@{0}`

## branch a 相对branch b临时做了一些变化，并提交了，在branch b上修改后，同步到branch a.

> 意味着 branch a 需要回退之前做的那些变化，再把branch b 的提交合并

1. 回退临时的提交
`git revert 59e34ec87b98393c6c1a5feb468b386a71177dfc`
2. 切换到branch b上,获取要合并的那个提交的commit id
`git checkout b`
3. 切换到branch a上，并合并
`git checkout a`
`git cherry-pick 7dc5492cdd163ab049f883d718524a43d62ab0eb`


## 从web上拉取了一段代码，然后合并到主干分支
`git fetch ssh://xiangsun@192.168.120.246:29418/GVC32XX/android/grandstream/apps refs/changes/93/205193/1 && git checkout FETCH_HEAD`

1. `git branch -v` 查看领先多少
2. `git branch temp` 新建临时分支
3. `git checkout temp` 切换到临时分支，把修改的内容合并到temp分支
4. `git checkout b` temp分支要合并到b分支中，切换到temp分支
5. `git merge temp` 合并temp分支
6. `git status` 解决冲突
7. `git branch -d temp` 删除temp分支

## git 更改名称
`git mv old_name new_name

## git 修改某一次提交的记录
- 修改最近一次
`git commit --amend`

- 修改前某一次提交（该提交id是 commit_id）
    1. git stash 或者第3步再修改。
        
    2. 回退到commit_id的提交，有以下几种方式
    
        ```shell
        git rebase -i GVC3220^ //GVC3220 是branch，^ 倒数第一个提交，^^倒数第二个提交
        git rebase -i GVC3220~n //表示倒数第n个提交
        git rebase -i commit_id^ //回退到该id
        ```
    
        此处会进入编辑界面，该编辑界面会把你选择的commit_id 到当前最新的提交倒序写出来，也就是commit_id会是第一个
    
    3. 修改上面编辑页面的`pick`为`edit`
    
    4. `git stash pop` 或者直接修改
    
    5. `git commit --amend`
    
    6. `git rebase --continue`
    

## 在git命令行下查看git stash里面文件的内容

在使用git的时候往往会保存一些东西，在保存的时候使用的就是git stash，强大的git使得保存修改和恢复修改变的很容易，但有时候时间久了不记得stash里面的内容是什么了，通过在stackflow里面查找，找到了一个好的方法。

在git bash上可以使用git --help stash来查看git stash命令的用法，当在stash后加show时，官方给出的介绍如下：

 

show [<stash>]

Show the changes recorded in the stash as a diff between the stashed state and its original parent. When no <stash> is given, shows the latest one. By default, the command shows the diffstat, but it will accept any format known to git diff(e.g., git stash show -p stash@{1} to view the second most recent stash in patch form). You can use stash.showStat and/or stash.showPatch config variables to change the default behavior.

翻译如下：显示修改在stash状态与原版本之间的不同变化。当没有<stash>给定时，显示最新stash的变化。默认情况下，命令显示diffstat，它可以接受任何已知的git diff格式（例如，git stash show -p stash@{1}是查看第二最近stash的变化）。你可以使用stash.showstat和/或stash.showpatch配置变量来改变默认的行为。也就是使用git stash show -p stash@{1}来查看stash的内容变化。


`git stash show -p`