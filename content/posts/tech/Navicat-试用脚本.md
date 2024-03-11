---
title: "Navicat 试用脚本"
date: 2024-02-02T15:29:04+08:00
lastmod: 2024-02-02T15:29:04+08:00
author: ["熊大如如"]
description: ""
tags:
  - "navicat"
weight:
slug: ""
summary: "试用 Navicat 16"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: ""  # 文章的图片
---

### windows 脚本
```shell
@echo off
set dn=Info
set dn2=ShellFolder
set rp=HKEY_CURRENT_USER\Software\Classes\CLSID
:: reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration14XCS /f  %针对<strong><font color="#FF0000">navicat</font></strong>15%
reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration16XCS /f
reg delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Update /f
echo finding.....
for /f "tokens=*" %%a in ('reg query "%rp%"') do (
 echo %%a
for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn%" /s /e ^|findstr /i "%dn%"') do (
  echo deleteing: %%a
  reg delete %%a /f
)
for /f "tokens=*" %%l in ('reg query "%%a" /f "%dn2%" /s /e ^|findstr /i "%dn2%"') do (
  echo deleteing: %%a
  reg delete %%a /f
)
)
echo re trial done!
  
pause
exit
```

### linux
```shell
rm -rf ~/.config/navicat    
rm -rf ~/.config/dconf/user
 
lsof | grep navicat | grep \\.config
```

### macos
```shell
# copy from https://github.com/pretend-m/navicat_for_mac_reset

rm -rf ~/Library/Preferences/com.navicat.NavicatPremium.plist

regex="\.([0-9A-Z]{32})"
[[ $(ls -a ~/Library/Application\ Support/PremiumSoft\ CyberTech/Navicat\ CC/Navicat\ Premium/ | grep '^\.') =~ $regex ]]

hash=${BASH_REMATCH[1]}

if [ ! -z $hash ]; then
    rm ~/Library/Application\ Support/PremiumSoft\ CyberTech/Navicat\ CC/Navicat\ Premium/.$hash
fi
```

