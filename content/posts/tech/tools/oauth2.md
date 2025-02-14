---
title: "Oauth2 入门"
date: 2025-02-14T20:04:56+08:00
lastmod: 2025-02-14T20:04:56+08:00
author: ["熊大如如"]
tags: # 标签
  - ""
description: ""
weight:
slug: ""
summary: "OAuth2 是一种行业标准的授权协议，包含一系列流程和标准。即授权第三方应用，获取用户数据。"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" # 文章的图片
---

## 一、OAuth2 是什么

OAuth2 是一种行业标准的授权协议，包含一系列流程和标准。即授权第三方应用，获取用户数据。

## 二、举例

你要开发一个微信小游戏，需要获取微信用户的好友信息，从而完成用户积分排名这一功能。

你如何获取用户的好友信息的同时不获取用户的敏感信息？

我们可以设计一套授权机制

- 小游戏向用户获取授权，用户同意或者拒绝
- 用户同意，并向小游戏颁发一个令牌
- 小游戏可以通过令牌获取用户的敏感信息，比如好友列表

**现在 OAuth 就是这套授权机制**

## 三、OAuth2 的优点

1）令牌有有效时间，比如七天有效期

2）令牌有权限范围（scope），比如只能读取某些数据

3）令牌可以被数据所有者撤销，会立即失效

上面这些设计，保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。这就是 OAuth 2.0 的优点。

## 四、OAuth2 的四种方式

- 授权码（authorization-code）
- 隐藏式（implicit）
- 密码式（password)
- 客户端凭证（client credentials）

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

**下面逐一介绍一下**

### 1.授权码（authorization code）方式

授权码方式，指的是向授权服务器获取一个授权码 code 然后再通过 code 获取令牌

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

**图示**

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502142006475.png)

#### 1) 获取授权码请求示例

```shell
HTTP/1.1 GET
/authorize?response_type=code&client_id=yourclientid&state=xxxx&redirect_uri=http://a.com/callback&scope=read
```

> response_type：必填，参数表示需要返回数据类型，code 为授权码类型
>
> client_id：必填，客户端标识，即第三方应用 id
>
> redirect_uri：可选，告诉应用认证平台的服务端认证通过/失败后需要跳转的地址
>
> state：可选，应用认证平台认证后重定向回客户端时包含此值（原封不动地返回），该参数可用于防止跨站点请求伪造
>
> scope：可选，允许第三方应用访问的请求范围。如此处 read 可以认为此第三方应用只有只读权限，不可写

#### <font style="color:rgb(37, 41, 51);">2) 认证平台授权通过后，跳回 redirect_uri 指定的地址</font>

```shell
HTTP/1.1 GET
http://a.com/callback?code=thisiscode&state=xxxx
```

#### <font style="color:rgb(37, 41, 51);">3）第三方应用拿到授权码后发起获取令牌请求</font>

```shell
HTTP/1.1 GET
/authorize?client_id=yourclientid&client_secret=your_c_secret&grant_type=authorization_code&code=thisiscode&redirect_uri=CALLBACK_URL
```

> client_id：客户端标识，即第三方应用 id
>
> client_secret：第三方应用 secret
>
> grant_type：说明是使用哪种授权方式来申请令牌的，authorization_code 为 code 方式
>
> code：上一步获取到的授权码
>
> redirect_uri：接收令牌的回调地址，可选

#### 4）授权网站收到请求以后，就会颁发令牌

```shell

{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

### 2. 隐藏式

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

**图示**

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502142007960.png)

### 3. 密码式

这种方式建立的前提是，你特别信任某个应用，信任到可以将自己的账号密码交给它。

它拿着你的账号密码，获取令牌。

```shell
https://a.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

### 4.凭证式

最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。

我直接 pass 了，不适用与我，有兴趣的可自行 Google 😅

## 五、如何使用令牌

获取令牌之后，在请求头中带上 token，访问应用的 API 即可

```shell
https://a.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

## 六、链接

- [OAuth 2.0 的一个简单解释](https://www.ruanyifeng.com/blog/2019/04/oauth_design.html)
- [OAuth 2.0 的四种方式](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)
- [什么是 OAuth2？](https://juejin.cn/post/7155058341168087076)

欢迎指正~
