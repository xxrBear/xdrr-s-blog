---
title: "Hugo 快速搭建部署静态博客教程"
description: "Hugo 快速搭建部署静态博客教程"
summary: "Hugo 快速搭建部署静态博客教程"
date: 2023-01-22T14:28:07+08:00
lastmod: 2023-12-11T14:28:07+08:00
author: [ "熊大如如" ]
tags: # 标签
  - "Hugo"
weight:
slug: "Hugo 快速搭建教程"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://raw.githubusercontent.com/xxrBear/image/master/Hugo202401131124538.png"
---


## 1.什么是 Hugo
Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

[Hugo官网](https://gohugo.io/)


## 2.如何下载 Hugo

<details>

<summary>点击查看</summary>

- Windows
```
winget install Hugo.Hugo.Extended
```
- MacOS
```
brew install Hugo
```
- Ubuntu/Debian
```
sudo apt install Hugo
```

</details>


## 3.创建 Hugo 博客项目
```
Hugo new site myblog
```

## 4.本地启动 Hugo 博客项目
- `Hugo`:会在项目文件夹下生成public文件
- `Hugo server`:启动Hugo自带的开发者服务器，可以用来预览效果
- `Hugo -F --cleanDestinationDir`:生成一个全新的public文件夹

## 5.本地预览 Hugo 博客项目
```angular2html
localhost:1313
```

## 6.新写一篇文章
```angular2html
hugo new content/posts/第一篇文章.md
```

## 7.部署到腾讯云服务器
> 演示环境基于 Ubuntu 22.04 LTS
> 
>默认您已经完成了前面六步，并生成了public文件夹

<details>

<summary>点击查看</summary>

### 1.安装 nginx 服务器
```angular2html
sudo apt install nginx
```
### 2.进入 nginx 配置文件所在的文件夹下
```editorconfig
cd /etc/nginx/
```
### 3.修改 nginx.conf 配置文件如下
```editorconfig
# 要配置的第一个地方，这里的用户要改成root，不然可能会没有权限
user root;

worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
    
    # 配置http
    server {
        # 要配置的第二个地方，80访问端口
        listen       80 default_server; 
        listen       [::]:80 default_server;
        
        # 要配置的第三个地方，域名
        server_name www.xxrbear.cn;
        rewrite ^(.*) https://$server_name$1 permanent; #自动从http跳转到https
        # IP 访问跳转至域名，将域名替换成自己的
        if ($host !~ (xxrbear.cn)$){
                rewrite ^ https://xxrbear.cn$request_uri?;
                }
        # IP 访问跳转至域名，将 IP 和域名替换成自己的
        if ($host ~ 122.51.41.171){
                rewrite ^ https://xxrbear.cn$request_uri?;
                }


        # 要配置的第四个地方，这里指向public文件夹
        root /home/ubuntu/myblog/public;

        include /etc/nginx/default.d/*.conf;
        
        # 要配置的第五个地方
        location / {
            root /home/ubuntu/myblog/public;
            index  index.html index.htm;
        }
        
        # 要配置的第六个地方
        error_page 404 /404.html;
        location = /40x.html {
            root   /home/ubuntu/myblog/public;
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    
    # 配置https的步骤，此处可以忽略
    server {
         listen 443 ssl;
         # 要配置的第七个地方
         server_name www.xxrbear.cn;
         root /home/ubuntu/myblog/public;
         
         # 要配置的第八个地方
         ssl_certificate /home/ubuntu/myblog/xxrbear.cn_nginx/xxrbear.cn_nginx/xxrbear.cn_bundle.crt;
         ssl_certificate_key /home/ubuntu/myblog/xxrbear.cn_nginx/xxrbear.cn_nginx/xxrbear.cn.key;
         
         # 要配置的第九个地方，可以按照我的写法
         ssl_session_timeout 10m;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
         ssl_prefer_server_ciphers on;
         
         # 要配置的第十个地方
         error_page 404 /404.html;
         location = /404.html {
              root /home/ubuntu/myblog/public;
         }

         include /etc/nginx/default.d/*.conf;
     }

}
```
### 4.测试 nginx.conf 配置文件是否正确
```shell
sudo nginx -t

# 显示类似如下输出则配置文件正确
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful 
```
### 5.预览效果
![我的个人博客](https://raw.githubusercontent.com/xxrBear/image/master/Hugo202401131124538.png)
</details>



## 更多参考
- [Sulv's Blog](https://www.sulvblog.cn/)
- [Hugo官方文档](https://goHugo.io/documentation/)
