# git
关于git的方方面面。包括基本概念，git flow，git提交规范，git插件以及常见问题解决
## 概念

> 不要死记命令，而要知道命令背后的原理，即命令背后究竟做了什么事情。

### git是一个分布式版本控制系统

说它是分布式，其实是相对于传统的集中式版本控制系统如SVN来说的。
git有本地仓库和远程仓库的概念。因此就涉及到了本地仓库和远程仓库的同步问题。

通过以下命令可以查看远程分支的地址信息：

```bash
git:(daily/0.1.0) git remote -v
origin  http://gitlab.test.com/f/test.git (fetch)
origin  http://gitlab.test.com/f/test.git (push)
```

本地分支拥有完整的内容，多人协作的时候，可以通过clone远程分支，通过fetch同步远程仓库，然后在本地仓库做修改，push 到远程达到多人协作的目的。

### git是面向对象，本质上是内容寻址系统

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

### 暂存区， 工作区，远程仓库的概念和区别

比如add，commit，checkout [file]， reset究竟是在三者中怎么变化的。

暂存区，工作区，远程仓库的关系可以用一张图来表示。为了方便，直接拿廖雪峰的图：

 ![workspace-stage](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)

git add 会添加文件到暂存区，commit是将暂存区的文件提交到版本库。 然后push才最终将代码推送到远程对应的分支。

另外git还有一个文件状态生命周期的概念，同样也可以用一张图来表示。我直接从git-scm拿的图:

![life-cycle](https://github.com/azl397985856/git/blob/master/illustrations/lifecycle.png)
### HEAD，指针

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
 通过如下命令可以查看当前HEAD位置：
 
 ```bash
 git:(develop) cat HEAD
ref: refs/heads/develop
```
 
 hooks是钩子。用来在git操作前后进行一些操作。 比如下面讲的husky插件就是基于这个原理实现的。
 
 index是本地的暂存区
 
 objects就是上面讲的内容
 
 refs其实是分支的引用。
 
 ```bash
 git:(daily/0.1.0) ✗ ls .git/refs
heads   remotes tags

git:(master) ✗ cat .git/refs/remotes/origin/HEAD
ref: refs/remotes/origin/master
```
## git flow
git flow其实就是分支管理模型。对于大型项目遵循一定的规则是很有必要的，
你可以定义一个完全适合你自己项目的工作流程，或者使用一个别人定义好的。

典型的git flow流程大概是这样的：

<img alt="git-flow" src="https://github.com/azl397985856/git/blob/master/illustrations/git-flow.png" width="650">

目前比较流行的git flow包含如下几种分支类型。
### hotfix
工作分支，用于修复线上bug
### feature
工作分支，用于开发新的功能
### develop
合并分支，用于合并hotfix和feature。 develop分支恐怕是我们用的最多的分支。
develop汇总了所有的功能分支代码。当develop代码达到稳定状态（测试完成），将其合并到release分支。

> develop其实包含了所有的功能分支，包括未被测试的分支。
当分支被测试完毕才可以将其合并到release。所有的分支只有master和develop是长期分支。
### release
 合并分支，用于发布某一个版本，通常采用semver。
 release分支通常不做任何代码修改的，仅仅修改版本号，构建信息等元数据。但是其提供了发布生产最后修改代码的机会，也就是
 说你可以正在这里进行`小范围`的代码修改。
 
### master
应该是随时可以发布的代码（这点非常重要），不可对其进行任何提交，只可以从hotfix或者release合并。

git flow是被应用广泛的一种分支管理模型，受到很多开发者的追捧。
但是其复杂性确并不被所有开发者买单，google就没有采用git flow的开发模式。
这有一篇文章，讲的就是git flow的反模式。[anti git-flow](http://endoflineblog.com/gitflow-considered-harmful)

如果合并到develop中的一个分支没有通过测试，无法发布，其他分支需要发布怎么办？

这就需要checkout develop，然后revert指定feature分支合并的commit id。这也就是为什么
git flow 合并分支需要非快速合并的原因（--no-ff）
 
 更详细的git flow的使用方法见文末的参考文献。
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
好的工具不仅可以提高效率，减少失误，配合起来使用更能达到意想不到的效果。
下面我推荐下我个人正在使用的一些git插件。

- [git](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git)
- [git-flow](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/git-flow)
- [commitzen](https://github.com/commitizen/cz-cli)
- [coventional-changelog](https://github.com/commitizen/cz-conventional-changelog)
- [husky](https://github.com/typicode/husky)
- [git blame](https://github.com/Sertion/vscode-gitblame) or [git lens](https://github.com/eamodio/vscode-gitlens)
 
## 常见场景及解决方案

> 前提是需要理解上面的基本概念

1. 如何回退某个提交的内容？

如果需要回滚某个提交，可以提交一个新的提交，将那次提交的内容给反向抵消掉。
这正是git revert commit-id 所做的。

2. 如何回退到某次提交？

理论上我们可以通过git revert HEAD ， git revert HEAD^1 ...
但是这样比较麻烦，你可以通过git reset commit-id完成。

这样操作和git revert是有区别的。 它直接将指针指向commit-id，强制改变了提交历史。
而不是创建新的提交，这样做会有风险，因此推送到远程的时候需要强制操作。 git push --force

> 如果git reset 之后想要回到reset之前的版本。可以通过git reflog查看，然后再次通过git reset commit-id回滚。
如果把git commit 看成是游戏存档的话， 那么git reflog就是存档记录

3. 我在开发一个功能，此时线上有个bug。但是功能写了一半，不能提交，怎么办？

可以通过git stash将工作区的内容存储起来。然后切换新分支完成bug修复，再次切换到未完成的分支，执行
git stash pop 将未完成的工作还原到工作区。

更多场景及解决方案可以[issue](https://github.com/azl397985856/git/issues) 或者 [pr](https://github.com/azl397985856/git/pulls) 给我。

## 参考资料
[git-flow](https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/git-flow)

[a-successful-git-branching-model](http://nvie.com/posts/a-successful-git-branching-model/)

[Git-Internals-Git-Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

[Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)

> 上述参考资料中git-flow 和 Git-Internals-Git-Objects 有对应中文版，可以直接修改url中的en为cn查看中文内容。
