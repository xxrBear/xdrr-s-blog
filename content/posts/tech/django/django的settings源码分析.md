---
title: "Django如何从settings中获取属性值"
date: 2024-03-29T13:58:00+08:00
lastmod: 2024-03-29T13:58:00+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
- "django"
description: ""
weight:
slug: ""
summary: "django读取settings模块属性值的源码简单分析"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image/202403291354659.jpg"  # 文章的图片
---

### 1.配置类的继承关系
<div style="text-alient: center">
<img src="https://cdn.jsdelivr.net/gh/xxrBear/image/202403291342737.png" alt="example" style="width: 180px; height: auto;">
</div>

`LazyObject`为顶级父类，`LazySettings`继承他。`Settings`类被`LazySettings`类所包裹。

### 2.Settings类的作用
[Settings源码](https://github.com/xxrBear/django-chinese-annotation/blob/master/django/conf/__init__.py)

查看Settings类源码，发现他接收一个变量 `settings_module`，先将 `global_settings` 的所有值设置为自己的实例属性，然后再将 `settings_module`的变量设置为实例属性。

当`settings_module`与`global_settings`的变量名重复时，以`settings_module`变量为主。

### 3. 特殊方法
在了解`LazySettings`类的作用之前，我们要先明白__dict__方法和__getattr__方法的作用。

简单来说，__dict__方法记录着类的属性字典。当使用`.`运算符时，首先会从类的属性字典里取值，即：


    self.xxx取得 self.__dict__['xxx']的值


当__dict__没有取到属性值时会运行到__getattr__方法内取属性值。

举例：

    class Animal:
        def __getattr__(self, attr):
            return 'xxx'
    
    a = Animal()
    a.xxx
    
    'xxx'

### 4.LazySettings类的作用
[LazySettings源码](https://github.com/xxrBear/django-chinese-annotation/blob/master/django/conf/__init__.py)

查看`LazySettings`的`_setup`方法，他将`self._wrapped`设置为`Settings`类实例。前面说了，`Settings`类里面有系统和自定义的所有的变量。所以当我们从`LazySettings`中获取属性时，会走到`LazySettings`的`__getattr__`方法中。

查看一下这个方法，你会发现，他从`self._wrapped`中获取属性，也就是从`Settings`类中获取属性，然后会将属性写入`__dict__`方法中，也就是说，下次获取属性会直接从`LazySettings`类的`__dict__`中直接获取属性，避免了每次加载配置文件，减少了内存开销。

### 5.问题

**问：Django为什么不直接从Settings中获取属性，而是从LazySettings中获取属性**

Django 使用 LazySettings 类来延迟加载设置属性的值的主要原因是为了在设置加载过程中提供更大的灵活性和可定制性。

1.延迟加载：LazySettings 使用延迟加载的机制，即在需要访问设置属性时才会加载对应的值。这样可以避免在 Django 启动时一次性加载所有设置，减少了启动时间。

2.动态修改：由于设置是延迟加载的，您可以在运行时动态修改设置。这使得在某些情况下可以根据需要调整设置，而无需重启应用程序。

3.可扩展性：LazySettings 类提供了许多方法和钩子，使得您可以在设置加载的过程中进行自定义操作，从而增加了 Django 框架的可扩展性。

总的来说，使用 LazySettings 类可以提供更灵活和可定制的设置加载机制，使得 Django 更适应不同的应用场景和需求。

### 6.声明
如有错误或不准确请指正，感谢！

