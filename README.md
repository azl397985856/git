# git
关于git的方方面面。包括基本概念，git flow，git提交规范，git插件以及常见问题解决
## 概念

- git是一个分布式版本控制系统

说它是分布式，其实是相对于传统的集中式版本控制系统如SVN来说的。
git有本地仓库和远程仓库的概念。因此就涉及到了本地仓库和远程仓库的同步问题。

通过以下命令可以查看远程分支的地址信息：

```bash
git:(daily/0.1.0) git remote -v
origin  http://gitlab.test.com/f/test.git (fetch)
origin  http://gitlab.test.com/f/test.git (push)
```

本地分支拥有完整的内容，多人协作的时候，可以通过clone远程分支，通过fetch同步远程仓库，然后在本地仓库做修改，push 到远程达到多人协作的目的。

- git是面向对象，本质上是内容寻址系统

.git目录的下有一个文件夹objects。存储了git库中的对象，对象是git中非常重要的概念。

```bash

git会对所有内容做hash-obejct生成40位的校验和，然后前两位作为文件夹的名字。如下：

git:(daily/0.1.0) ✗ ls -1 .git/objects
00
01
02
03
04
05
06

后38位作为文件名：

git:(daily/0.1.0) ✗ ls -1 .git/objects/00
04a9fe24ed3e25e7a9d0cd87178bc4aed32891
1ab1dbd031b06b88acf247c2734a6ce5f2c2f2
384c5dbd2a126e09f27aa3f818516830b38933
604011e29363914659a5e436564f0c20b95ff4
80c38f950f9199744b82328f29ceb8ba5e7745
98a6385296048f8b3e5bf923374266f5df3d36
b876a0e0adfa03a51862b0cbc7e04015628257
bf64fbe6d61edbcc43484cd1feebf48ffbc668
ff7645be03614527ec8c15f943e6d77ed56749

可以通过如下命名查看具体的文件内容,下图是显示tree的内容，如果显示blob内容，则会直接显示文件内容本身。

可以通过 git cat-file -t hash 查看对象类型

git:(daily/0.1.0) ✗ git cat-file -p 001ab1dbd031b06b88acf247c2734a6ce5f2c2f1
100644 blob 4b02ba48a1dc2be18735b763d7e17c8aa9640ac5    index.js
100644 blob 7c49e625833ea4009c035ac8a568b53b0b68a3da    a.less
100644 blob b1c19145a728e038bfb9b94525d48f6df70fb82f    b.vue

```

git就是根据object建立一种树形结构。将文件和通过hash的方式关联起来。

当你推送远程分支的时候会看到类似下面的信息：

```bash
git:(daily/0.1.0) ✗ git gc
Counting objects: 843, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (764/764), done.
Writing objects: 100% (843/843), done.
Total 843 (delta 453), reused 0 (delta 0)
```
你也可以像我一样用git gc手动触发。

- 暂存区， 工作区，远程仓库的概念和区别

比如add，commit，checkout [file]， reset究竟是在三者中怎么变化的。

- HEAD，指针

> 不要死记命令，而要知道命令背后的原理，即命令背后究竟做了什么事情。

当你新建一个git项目的时候，git会在根目录新建一个git目录.

git 目录的文件内容：

```bash
➜  .git git:(master) ls -F1
COMMIT_EDITMSG
HEAD
ORIG_HEAD
config
description
hooks/
index
info/
logs/
objects/
packed-refs
refs/
```
 其中比较重要的是HEAD hooks  index objects refs这五个，理解上面五个概念对理解git非常重要。
 
 HEAD是一个特殊的指针，它是指针的指针。它用来标记当前的提交。当你使用git checkout branch，
 HEAD指针就会发生移动。 当你新建一个分支的时候其实仅仅是改变了HEAD的指向，这也是git分支比较轻量的原因。
 
 hooks是钩子。用来在git操作前后进行一些操作。 比如下面讲的husky插件就是基于这个原理实现的。
 
 index是本地的暂存区
 
 objects就是上面讲的内容
 
 refs其实是分支的引用。
 
 ```bash
 git:(daily/0.1.0) ✗ ls .git/refs
heads   remotes tags
```
## git flow
你可以定义一个完全适合你自己项目的工作流程，或者使用一个别人定义好的。

目前比较流行的git flow包含如下几种分支类型。
### hotfix
工作分支，用于修复线上bug
### feature
工作分支，用于开发新的功能
### develop
合并分支，用于合并hotfix和feature
### release
 合并分支，用于发布某一个版本，通常采用semver
## git commit msg
好的提交不仅方便查看，由于其一致的数据格式，还可以用于其他处理的数据源。
如根据commit msg，生成changelog。

下面介绍一种被广泛使用的提交规范
### angular commit message conventions

由必选的几个部分和可选的部分组成的简短的git提交信息规范。

形如：

```
feat: 增加一个新特性
fix(module-a): 修复一个bug #89123
doc: 增加文档
chore: 增加注释

```
详细介绍可以参考文末的参考资料。
## 插件

- [git](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git)
- [git-flow](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/git-flow)
- [commitzen](https://github.com/commitizen/cz-cli)
- [coventional-changelog](https://github.com/commitizen/cz-conventional-changelog)
- [husky](https://github.com/typicode/husky)
- [git blame](https://github.com/Sertion/vscode-gitblame) or [git lens](https://github.com/eamodio/vscode-gitlens)
 
## git 常见问题

> 需要理解上面的基本概念

1. 如何回退某个提交的内容？
2. 如何回退到某次提交？
3. 多人协作的回退版本，如何保证不影响其他成员？
4. 我在开发一个功能，此时线上有个bug。但是功能写了一半，不能提交，怎么办？

## 参考资料
[git-flow](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/git-flow)

[Git-Internals-Git-Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

[Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)
