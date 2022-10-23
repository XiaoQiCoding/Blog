嗨咯，大家好我是小棋，上次教大家如何在本地用Git管理自己的项目，这次我们一起来了解下Git远程仓库：github。

关注我，一起进步吧~
- [bilibili 打工人小棋](https://space.bilibili.com/302482063?spm_id_from=333.1007.0.0)

- [知乎 打工人小棋](https://www.zhihu.com/people/jin-tian-ye-yao-kai-xin-ya-58-32)

- [CSDN 打工人小棋](https://blog.csdn.net/dagongrenxiaoqi?spm=1000.2115.3001.5343)

# 前言
使用Git管理项目让我们事半功倍，但是把项目存放在本地是危险的，而且不利于多人协作，因此我们需要将项目push到远程仓库（可以类比百度云盘），用它来帮助我们托管项目。

我这里并不打算用过多的语言描述git中关于远程仓库的指令，而是翻译下github为我们提供的一个远程文档。这个文档是每次在github中创建项目之后自动生成的，文档虽然很短，但是包含了最重要的一些信息。理论上，只要知道这个文档的内容，就足够开始git远程开发了。

下面我对这个文档进行翻译，并加入一些我个人的解读。

# 原文
## Quick setup — if youʼve done this kind of thing before

Get started by creating a new file or uploading an existing file. 

We recommend every repository include a README, LICENSE, and .gitignore.

## ...or create a new repository on the command line
echo "# PVZ_Course" >> README.md

```
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:IMUHERO/PVZ_Course.git
git push -u origin main
```

## ...or push an existing repository from the command line

```
git remote add origin git@github.com:IMUHERO/PVZ_Course.git
git branch -M main
git push -u origin main
```

# ...or import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project

# 翻译和解读
## 开始：请确保做了下述的事情

在github网页中创建一个新的文件，或者上传本地已经存在的文件。
> 我的理解：github网页版仓库中有个按钮是`Add File`，因此我们可以通过网页版去创建新的文件。或者通过 git push 指令把本地的文件上传到github中。

我们建议，每一个项目仓库包含：README.md，LICENSE.md 和 .gitignore 文件。

> 我的补充：
- `.md`文件是markdown文档的文件格式，很多程序员都会使用这种格式书写文本，因为他简洁易读。
- `README.md`:是现在大多数项目介绍文档的统一命名，命名为这个文档的内容，会展示在github首页中，作为该项目的一个介绍。
- `LICENSE.md`:如果没有这个LICENSE，那么你的项目默认为独家版权，别人是不能碰你的项目的。
只有当你创建了LICENSE，并在LICENSE之中声明了开源许可证之后，你的项目才是开源的。

##  ...或者使用命令行创建一个项目仓库

使用到的指令如下：
```
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:IMUHERO/PVZ_Course.git
git push -u origin main
```

解读：没啥可解读的，就是打开一个文件夹，一个一个指令输入，就建立了这个本地文件夹和远程仓库的连接了！

## ...或者使用命令行提交已经创建好的项目仓库

如果你之前已经在本地使用git管理项目了，那么可以省去创建的步骤，直接建立和远程仓库的连接，并且提交你本地的这些问价以及修改记录。

```
git remote add origin git@github.com:IMUHERO/PVZ_Course.git
git branch -M main
git push -u origin main
```

# ...或者导入另一个项目仓库

...

一般不会用吧，不用管这一点了。

> 值得一说的是：当你在github中浏览别人的项目时，可以使用他项目右上角的fork按钮，把他的项目克隆一份到你自己的仓库中。因为你无法对他的项目做修改，但是拷贝完后，你就可以修改自己fork后的远程项目了~