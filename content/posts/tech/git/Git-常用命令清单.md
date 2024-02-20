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


### 一、添加文件
```shell
# 添加当前文件夹下的所有文件
git add .

# 添加修改的文件，不包括新增的文件
git add -u

```

### 二、提交
```shell
# 提交消息到仓库区
git commit -m ''

# 修改最近一次的提交
git commit --amend

```

### 三、分支
```shell
# 新建一个分支
git branch [new_branch]

# 新建一个分支并切换到这个新分支
git switch -c [new]

# 新建一个空分支
git switch --orphan [branch]

# 删除分支
git branch -d [branch]

# 切换分支
git switch [branch]

# 分支重命名
git branch -m [old] [new]

# 列出本地所有分支
git branch

```

### 四、远程分支
```shell
# 列出远程所有分支
git branch -r

# 拉取远程分支到本地的一个新分支
git switch -c [branch] origin/[branch]

```

### 五、日志
```shell
# 查看所有提交信息
git log

# 用户名为组显示提交记录
git shortlog

# 各用户提交数量，降序排列
git shortlog -n -s
```

### 六、追责
```shell
# 查看文件每行的提交信息
git blame [file]

```

### 七、暂存
```shell
# 暂存当前分支的修改
git stash

# 暂存当前分支的修改，包括未追踪文件
git stash -u

# 将最后一次暂存，应用到当前分支
git stash apply

# 删除最近一个暂存，并应用到当前分支
git stash pop

# 列出所有的暂存记录
git stash list

# 删除所有的暂存
git stash clear

```

### 八、差异
```shell
# 查看工作区与暂存区的差异
git diff

# 查看暂存区的差异
git diff --cached

```

### 九、远程同步
```shell
# 拉取远程分支的提交
git pull origin [branch]

# 推送本地分支到远程
git push origin [branch]

# 添加一个远程仓库的追踪
git remote add [shortname] [url]


```

### 十、克隆
```shell
# 克隆一个远程仓库(默认主分支)
git clone [url]

# 克隆一个远程仓库，自定义分支
git clone [url] -b [branch] [url]

# 克隆一个远程仓库，只保留最后一次提交
git clone [url] --depth=1 [url]

# 克隆一个远程仓库，包括子模块
git clone [url] --recurse-submodules

```

### 十一、配置
```shell
# git配置文件位置
Git/etc/config     # system
vim ~/.gitconfig   # global
vim .git/config    # local

# 设置用户名邮箱
git config [--global] user.name yourname
git config [--global] user.email youremail

# 设置别名
git config [--global] alias.st status

# 取消别名
git config [--global] --unset alias.st

```

### 十二、文件移动与重命名
```shell
# 移动
git mv file dir/

# file1重命名为file2
git mv file1 file2

```

### 十三、文件删除
```shell
# 删除文件并添加至索引区
git rm [file]

# 删除文件并将文件设置为未追踪
git rm --cached [file]

```

### 十四、文件撤销
```shell
# 暂存区的文件撤销到工作区
git restore --staged [file]

# 工作区的文件撤销到仓库区(丢弃修改)
# (此命令对暂存区的文件不生效)
git restore [file]

```

### 十五、版本回退
```shell
# 回退操作
git reset [mode] [commit]

# 回退到指定版本，修改还在暂存区
git reset --staged [commit]

# 回退到指定提交，修改还在工作区
git reset [commit]

# 回退到指定提交，三区一致
git reset --hard [commit]

```

### 十六、状态
```shell
# 查看三区状态
git status

```

### 十七、子模块
```shell
# 添加子模块到指定文件夹
git submodule add [url] [dir]

```

### 十八、变基
```shell
# 将代码变基到指定分支
git rebase [branch]

# 交互式变基
git rebase -i HEAD~num

```

### 十九、标签
```shell
# 增加标签
git tag [name]

# 删除标签
git tag -d [name]

# 修改标签名字
mv .git/refs/tags/1.9.1 .git/refs/tags/v1.9.1

# 列出所有标签
git tag

```

### 二十、挑选提交
```shell
# 挑选指定的提交
git cherry-pick [commit]

# 挑选指定的多个提交
git cherry-pick [commit1] [commit2]

```
