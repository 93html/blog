---
title: Github Fork 之后更新版本
date: 2017-6-26 22:00:07
categories: 技术
tags: Git
---
## 问题
比如说有一个 repo 叫做 a/project，现在被 b 用户 fork 了一份 b/project

当 a 更新了版本之后，b 如何获取到这份更新呢？

## 原理
Git 可以为仓库添加多个 remote，b 用户除了自己的 remote 外，再添加一个 a/project 的原 remote

然后在 pull 的时候指定到这个 a/project 的 remote 上就可以了

## 命令行操作
1. 查看当前 remote
```
$ git remote -v
origin  https://github.com/b/project.git (fetch)
origin  https://github.com/b/project.git (push)
```

2. 添加上游 remote
```
$ git remote add upstream https://github.com/a/project.git
$ git remote -v
origin  https://github.com/b/project.git (fetch)
origin  https://github.com/b/project.git (push)
upstream        https://github.com/a/project.git (fetch)
upstream        https://github.com/a/project.git (push)
```

3. 获取上游的分支和提交，master 上的提交会存在本地分支 `upstream/master`
```
$ git fetch upstream
```

4. 切换到 master 分支
```
$ git checkout master
```

5. 与 `upstream/master` 合并
```
$ git merge upstream/master
```

## SourceTree 操作
1. 打开 “仓库-项目设置-添加”，输入 a/project 的地址
2. “拉取-从远端拉取：upstream-要拉取的远端分支：master”

## 脑洞
那 b 用户岂不是可以直接 push 到 a/project ？？！
```
$ git push origin master
remote: Permission to a/project.git denied to b.
fatal: unable to access 'https://github.com/a/project.git/': The requested URL returned error: 403
Pushing to https://github.com/a/project.git
```
git 会很科学的返回 403 错误，优秀！
