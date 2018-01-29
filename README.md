# git
关于git的介绍，包括基本概念，git flow，git提交规范，git插件以及常见问题解决
## 概念

- git是一个分布式版本控制系统。
- git是面向对象的。本质上是内容寻址系统
- 暂存区， 工作区，远程仓库
比如add，commit，checkout [file] reset究竟是在三者中怎么变化的。
- HEAD，指针

> 不要死记命令，而要知道命令背后的原理，即命令背后究竟做了什么事情。

.git 目录的文件内容：

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
 
## git flow
你可以定义一个完全适合你自己项目的工作流程，或者使用一个别人定义好的。
### hotfix
工作分支，用于修复线上bug
### feature
工作分支，用于开发新的功能
### develop
合并分支，用于合并hotfix和feature
### release
 合并分支，用于发布某一个版本，通常采用semver
## git commit msg

### angular commit message conventions

由必选的几个部分和可选的部分组成的简短的git提交信息规范。

形如：

```
feat: 增加一个新特性
fix(module-a): 修复一个bug #89123
doc: 增加文档
chore: 增加注释

```

## 插件

- git
- git-flow
- commitzen
- coventional-changelog
- husky
- git blame or git lens
 
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
