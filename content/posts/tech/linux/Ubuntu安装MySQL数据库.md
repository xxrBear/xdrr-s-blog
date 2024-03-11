---
title: "Ubuntu安装MySQL数据库"
date: 2023-02-05T18:01:41+08:00
lastmod: 2023-02-05T18:01:41+08:00
author: ["熊大如如"]
keywords: 
-
tags: # 标签
    ["MySQL", "linux"]
description: ""
weight:
slug: ""
summary: "Ubuntu下安装MySQL8的简单步骤"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-mysql徽标-150.png"
---

### 1.安装MySQL服务

    sudo apt update
    sudo apt install mysql-server

### 2.MySQL安装脚本

    sudo mysql_secure_installation
    -- 要记住你的密码

### 3.修改root用户密码

    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'very_strong_password';
    FLUSH PRIVILEGES;

### 4.如果root用户密码修改失败,需要修改MySQL密码强度

```
SHOW VARIABLES LIKE 'validate_password%'; ---查看当前安全变量值


--修改密码策略
set global validate_password.policy=0;
set global validate_password.length=4;  -- 最低四位

重新改密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '1234';

```

### 5.如果忘记密码，修改密码

*   修改配置文件
    ```
    sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
    ```
*   末尾添加
    ```
    skip-grant-tables
    ```
*   MySQL重启
    ```
    system restart mysql
    ```
*   将旧密码置空
    *   执行mysql -u root -p登录数据库
    *   `use mysql`
    *   `update user set authentication_string = '' where user = 'root';`
    *   删掉步骤1的语句 `skip-grant-tables`
    *   重启服务 `systemctl restart mysql`
*   修改密码
    *   `mysql -uroot -p`
    *   `use mysql;`
    *   `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '1234';`
    *   失败还是需要修改密码强度

