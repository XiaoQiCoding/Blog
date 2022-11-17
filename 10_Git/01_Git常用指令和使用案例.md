嗨咯，大家好我是小棋，最近我出了一期关于Git管理Unity项目的视频，有很多小伙伴问我该如何学习Git。

因此，今天我就来系统的梳理下Git的学习路径、常用指令和使用案例。


关注我，一起进步吧~
- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

# 查看git帮助文档
工欲善其事，必先利其器，学会查看帮助文档胜过千言万语。
```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```
例如，要想获得 `git config` 命令的手册，执行

```
$ git help config
```

此外，如果你不需要全面的手册，只需要可用选项的快速参考，那么可以用 `-h` 选项获得更简明的 “help” 输出：

```
$ git add -h

usage: git add [<options>] [--] <pathspec>...

    -n, --dry-run         dry run
    -v, --verbose         be verbose

    -i, --interactive     interactive picking
    -p, --patch           select hunks interactively
    -e, --edit            edit current diff and apply
    -f, --force           allow adding otherwise ignored files
    -u, --update          update tracked files
    --renormalize         renormalize EOL of tracked files (implies -u)
    -N, --intent-to-add   record only the fact that the path will be added later
    -A, --all             add changes from all tracked and untracked files
    --ignore-removal      ignore paths removed in the working tree (same as --no-all)
    --refresh             don't add, only refresh the index
    --ignore-errors       just skip files which cannot be added because of errors
    --ignore-missing      check if - even missing - files are ignored in dry run
    --chmod (+|-)x        override the executable bit of the listed files

```

# 基础概念

git 报错或者提示信息中包含的一些基础名词，了解这些名词可以帮助你更快 debug

`Untracked files:`

不可追踪的文件，尚未git add 的文件

`stage:`

暂存区，git add 的时候存在这里

# 常用网站和资料

常用的一些网站和资料，本文也从中得到了些许参考

官网：[Git - Documentation](https://git-scm.com/doc)

廖雪峰git教程：[Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

常用git指令(案例)：[https://liaoxuefeng.gitee.io/resource.liaoxuefeng.com/git/git-cheat-sheet.pdf](https://liaoxuefeng.gitee.io/resource.liaoxuefeng.com/git/git-cheat-sheet.pdf)

# 常用指令
1. 初始化 git 本地仓库
```
git init
```

2. 查看状态
```
git status
```

3. 显示从最近到最远的提交日志
```
git log
```
4. 以单行模式显示log
```
git log --pretty=oneline
```
5. 删除工作区的修改到暂存区域，类似 git add
```
git rm
```
6. 回退命令
```
git reset
```
7. 回退到当前版本(回退 git add 的提交)
```
git reset HEAD file_name  
```
8. 切换到另一个分支
```
git checkout
```
git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

9. 撤销工作区的修改
```
- git checkout -- file_name
```

10. 切换到 master 分支
```
- git checkout master  
```
11. 记录每一次命令
```
git reflog
```

12. 查看工作区和版本库里面的版本区别
```
git diff
```
- `git diff HEAD -- readme.txt` 查看工作区和版本库里面最新版本的区别

13. 远程操作
```
git remote 
```

14. 添加远程仓库
```
`git remote add [locate]` 
```
- git remote add origin git@github.com:michaelliao/learngit.git

15. 其他远程操作
> git remote
- `git remote -v` 查看远程库信息
- `git remote rm origin` 删除远程分支(解除本地和远程的绑定关系)
> git push
- `git push -u origin master`  推送本地master分支到远程，并且建立两边的关联
- `git push origin <tagname>`  推送标签到远程
- `git push origin --tags`     一次性推送所有tag到远程
git clone

> 从远程克隆文件到本地库
```
$ git clone git@github.com:michaelliao/gitskills.git
Cloning into 'gitskills'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
Receiving objects: 100% (3/3), done.
```

16. 切换/创建分支
```
git switch
```
- `git switch -c dev`  创建并切换到 dev 分支
- `git switch master` 直接切换到已有的master分支
git branch
> 分支操作
- `git branch` 查看当前分支
- `git branch -d dev`  删除 dev 分支
- `git branch --set-upstream-to <branch-name> origin/<branch-name>`  建立本地分支和远程分支的关联
- `git checkout -b branch-name origin/branch-name`  在本地创建和远程分支对应的分支

17. 合并dev分支到当前分支

git merge [dev]
// no fast_forward 模式，即merge dev分支，并且带有dev分支信息
```git
git merge --no-ff -m "merge with no-ff" dev
```

18. 暂时隐藏当前工作目录
git stash
- `git stash list` 查看隐藏的工作现场列表
- `git stash apply` 恢复现场，但不从列表中移除
```git
git stash apply stash@{0}
```
- `git stash drop` 移除列表中的stash
- `git stash pop`  恢复现场，并且从列表中移除

19. 将master分支上的某次提交，合并到当前dev分支
```
git cherry-pick

$ git cherry-pick <commit>
$ git branch
* dev
  master
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

20. 标签
git tag
- `git tag` 查看当前tag
- `git tag v1.0` 给当前提交打上标签
- `git tag v0.9 f52c633`  给指定提交打上标签
- `git tag -a v0.1 -m "version 0.1 released" 1094adb`  用`-a`指定标签名，`-m`指定说明文字
- `git tag -d v0.1` 删除标签
git show
 > 查看标签详情
 - `git show <tagname>`

---

关注我，一起进步吧~

- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

![点个赞叭~](https://raw.githubusercontent.com/XiaoQiCoding/Blog/main/Image/Zan3.jpg)


