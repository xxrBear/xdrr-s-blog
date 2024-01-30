---
title: "Windows 配置 Oh My Posh 美化终端"
date: 2024-01-30T15:49:07+08:00
lastmod: 2024-01-30T15:49:07+08:00
author: ["熊大如如"]
description: ""
weight:
slug: ""
summary: "Windows 配置 Oh My Posh 美化终端的步骤"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

### 1.下载配置oh-my-posh 
```shell
# 下载oh-my-posh
winget install JanDeDobbeleer.OhMyPosh -s winget

# 配置oh-my-posh的环境变量
$env:Path += ";C:\Users\user\AppData\Local\Programs\oh-my-posh\bin"
```

### 2.创建oh-my-posh的配置文件
```shell
# 检查 $PROFILE 文件是否存在, 如果不存在则新建空文件
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }

# 打开配置文件
notepad $PROFILE
```

### 3.配置oh-my-posh的配置文件
```shell
# 写入配置文件的内容
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\bubbles.omp.json | Invoke-Expression
```

### 4.配置oh-my-posh的主题
- windows上的配置文件中主题文件的位置: 
    
```
C:\Users\<UserName>\AppData\Local\Programs\oh-my-posh\themes
```

- 选择你喜欢的主题，通过第三步修改主题的名字

    例如：

```
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\M365Princess.omp.json | Invoke-Expression
```


### 5.问题与解决方案
-   XXX 在此系统上禁止运行脚本

输入:

```shell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

### 6.相关参考
- [官方文档](https://ohmyposh.dev/docs/installation/windows)
- [Windows Terminal 美化 / PowerShell 美化: oh-my-posh 主题安装和使用](https://blog.csdn.net/Likianta/article/details/124950605)
