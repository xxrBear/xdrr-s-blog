---
title: "Go 语言指针"
date: 2025-07-16T11:05:27+08:00
lastmod: 2025-07-16T11:05:27+08:00
author: ["熊大如如"]
tags: # 标签
  - "go"
summary: "简单介绍Go语言的指针"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: ""  # 文章的图片
---
## 简介
Go 语言的指针的含义是：**记录某个变量内存地址的变量**

```go
var a int = 42
var p *int = &a
```

p 就是指针变量

```go
fmt.Println(p)  // 输出地址，例如：0xc000014088
fmt.Println(*p) // 输出 42
```

*p 意思是解引用

## 基本操作
### 获取内存地址
```go
a := 100
p := &a
```

### 解引用
```go
fmt.Println(*p) // 100
*p = 200
fmt.Println(a)  // 200
```

### 空指针判断
```go
var p *int
if p == nil {
    fmt.Println("p is nil")
}
```

## 引用传递
Go 语言中函数的参数默认是值传递，如果想要修改原始值就必须使用指针

```go
func changeValue(p *int) {
    *p = 100
}

func main() {
    a := 10
    changeValue(&a)
    fmt.Println(a) // 输出 100
}
```

