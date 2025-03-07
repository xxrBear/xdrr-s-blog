---
title: "一文粗通JWT：从原理到实战"
date: 2025-03-07T12:16:27+08:00
lastmod: 2025-03-07T12:16:27+08:00
author: ["熊大如如"]
tags: # 标签
  - "jwt"
description: ""
weight:
slug: ""
summary: "一文粗通JWT：从原理到实战 FastAPI 实现 Token"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202503071224472.jpg" # 文章的图片
---

## 一、简介

### 1.1 token 的验证流程

**让我们先来回顾一下 token 的验证流程**

1. 浏览器使用账号密码登录
2. 服务端收到请求，验证用户名和密码
3. 验证成功后，服务端会签发一个 token，再把这个 token 返回给客户端
4. 浏览器收到 token 后可以把它存储起来，比如放到 cookie 中
5. 浏览器每次向服务端请求资源时需要携带服务端签发的 token，可以在 cookie 或者 header 中携带
6. 服务端收到请求，然后去验证浏览器请求里面带着的 token，如果验证成功，就向客户端返回请求数据

如果您对 Oauth2 验证流程感兴趣的话，可以看下博主的这篇文章

[Oauth2 授权流程入门](https://www.xxrbear.cn/posts/tech/tools/oauth2/)

token 与 session 和 cookie 相比，主要优秀在以下几点：

1. 支持跨域访问：cookie 是无法跨域的，而 token 由于没有用到 cookie(前提是将 token 放到请求头中)，所以跨域后不会存在信息丢失问题
2. 无状态：token 机制在服务端不需要存储 session 信息，因为 token 自身包含了所有登录用户的信息，所以可以减轻服务端压力
3. JWT 是跨语言的，原则上支持任何语言的实现

### 1.2 什么是 **JWT**

**JWT**（JSON Web Token） 是一种生成 Token 的标准方式。它是一种开放标准（RFC 7519），用于在网络应用环境间安全地传递信息（通常用于身份验证和信息交换）。JWT 生成的 Token 可以被用作身份验证凭证，客户端在请求时携带这个 Token，服务器通过验证 Token 来确认用户的身份和权限。

**即 JWT 就是上述流程当中生成 token 的一种具体实现方式**

## 二、JWT 结构

JWT 由三部分组成，用点（`.`）分隔：

1. Header（头部）
2. Payload（负载）
3. Signature（签名）

```shell
Header.Payload.Signature
```

### 2.1 Header

Header 通常由两部分组成：

- alg：签名算法，如 HMAC SHA256 或 RSA
- typ：令牌类型，通常是 "JWT"

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2.2 Payload

负载，是 JWT 的主体内容部分，也是一个 JSON 对象，包含需要传递的数据。 JWT 指定七个默认字段供选择

```plain
iss：发行人
exp：到期时间
sub：主题
aud：用户
nbf：在此之前不可用
iat：发布时间
jti：JWT ID用于标识该JWT
```

### 2.3 Signature

Signature 是 JWT（JSON Web Token）的核心部分，用于确保 Token 的完整性和真实性。签名的作用是防止 Token 被篡改，确保只有持有正确密钥的服务器才能生成和验证 Token。

**Signature 的实现：**

1. 将 Header 和 Payload 分别进行 base64 编码，得到两个字符串
2. 将编码后的 Header 和 Payload 用 `.` 连接起来，形成一个字符串
3. 使用指定的签名算法和 **密钥**，对拼接后的字符串进行加密，生成 Signature
4. 将 Header、Payload 和 Signature 用 `.` 连接起来，组成最终的 JWT

**Signature**

**验证过程：**

1. 解析 JWT：

- 将 JWT 按照 `.` 分隔符拆分成 Header、Payload 和 Signature 三部分

2. 重新计算签名：

- 使用相同的算法和密钥，对 Header 和 Payload 重新计算签名

3. 比较签名：

- 将重新计算的签名与 JWT 中的 Signature 部分进行比较。如果一致，则说明 Token 未被篡改

## 三、实战

### 3.1 来个图

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202503071220903.png)

### 3.2 FastAPI 实现

让我们用一个 FastAPI 应用来实战一下

```shell
uv add pyjwt python-multipart # 安装pyjwt python-multipart
```

### 3.3 先实现一个伪数据库

```python
from pydantic import BaseModel
from typing import Optional

# 用户模型
class User(BaseModel):
    username: str
    password: str

# 模拟数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",  # 密码是 "secret"
    }
}
```

### 3.4 实现一个登录接口

```python
from pydantic import BaseModel
from typing import Optional

# 用户模型
class User(BaseModel):
    username: str
    password: str

# 模拟数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "password": "secret",  # 实际应用中应使用哈希密码
    }
}

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
import jwt
from datetime import datetime, timedelta

# JWT 配置
SECRET_KEY = "your-secret-key"  # 替换为你的密钥
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# OAuth2 配置
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()

# 获取用户
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return User(**user_dict)

# 认证用户
def authenticate_user(db, username: str, password: str):
    user = get_user(db, username)
    if not user:
        return False
    if user.password != password:  # 实际应用中应使用哈希密码验证
        return False
    return user

# 生成 JWT
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# 登录接口
@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}
```

### **3.5 实现受保护接口**

```python
from pydantic import BaseModel
from typing import Optional


# 用户模型
class User(BaseModel):
    username: str
    password: str


# 模拟数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "password": "secret",  # 实际应用中应使用哈希密码
    }
}

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
import jwt
from datetime import datetime, timedelta

# JWT 配置
SECRET_KEY = "your-secret-key"  # 替换为你的密钥
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# OAuth2 配置
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()


# 获取用户
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return User(**user_dict)


# 认证用户
def authenticate_user(db, username: str, password: str):
    user = get_user(db, username)
    if not user:
        return False
    if user.password != password:  # 实际应用中应使用哈希密码验证
        return False
    return user


# 生成 JWT
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


# 登录接口
@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


# 验证 JWT
def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token has expired",
            headers={"WWW-Authenticate": "Bearer"},
        )
    except jwt.InvalidTokenError:
        raise credentials_exception
    user = get_user(fake_users_db, username=username)
    if user is None:
        raise credentials_exception
    return user


# 受保护接口
@app.get("/users/me")
def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

登录 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 尝试一下

## 四、参考链接

- [JWT 详解](https://baobao555.tech/archives/40#1.%E4%BB%80%E4%B9%88%E6%98%AFjwt)
- [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
