---
title: "Git 常用命令清单"
date: 2024-01-31T13:10:22+08:00
lastmod: 2024-01-31T13:10:22+08:00
author: ["熊大如如"]
slug: ""
summary: "Git 常用命令清单"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image/202401311328129.png"  # 文章的图片
---


### 添加文件
```shell
# 添加当前文件夹下的所有文件
git add .

```

### 提交
```shell
# 提交消息到仓库区
git commit -m ''

# 修改最近一次的提交
git commit --amend

```

### 分支
```shell
# 新建一个分支
git branch [new_branch]

# 新建一个分支并切换到这个新分支
git checkout -b [new]
git switch -c [new]

# 删除分支
git branch -d [branch]

# 切换分支
git switch [branch]

# 分支重命名
git branch -m [old] [new]

# 列出本地所有分支
git branch

```

### 远程分支
```shell
# 列出远程所有分支
git branch -r

# 拉取远程分支到本地的一个新分支
git switch -c [branch] origin/[branch]

```

### 日志
```shell
# 查看所有提交信息
git log

```

### 追责
```shell
# 查看文件每行的提交信息
git blame [file]

```

### 暂存
```shell
# 暂存当前分支的修改
git stash

# 将最后一次暂存，应用到当前分支
git stash apply

# 删除最近一个暂存，并应用到当前分支
git stash pop

# 列出所有的暂存记录
git stash list

# 删除所有的暂存
git stash clear

```

### 差异
```shell
# 查看工作区与暂存区的差异
git diff

# 查看暂存区的差异
git diff --cached
```

### 远程同步
```shell
# 拉取远程分支的提交
git pull origin [branch]

# 推送本地分支到远程
git push origin [branch]

# 添加一个远程仓库的追踪
git remote add [shortname] [url]


```

