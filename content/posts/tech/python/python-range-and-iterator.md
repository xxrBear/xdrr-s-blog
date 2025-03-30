---
title: "[翻译] Python 中的 range 不是迭代器"
date: 2025-03-30T12:25:38+08:00
lastmod: 2025-03-30T12:25:38+08:00
author: ["熊大如如"]
description: ""
weight:
slug: ""
summary: "[翻译] Python 中的 range 不是迭代器"
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

原文链接：[Python: range is not an iterator](https://treyhunner.com/2018/02/python-range-is-not-an-iterator/)

在我参加 PyGotham 2017 演讲 Loop Better 时，有人问了我一个非常好的问题：

> 迭代器是惰性可迭代的，而 Python 3 中的 range 也是惰性可迭代对象，那它是迭代器吗？

遗憾的是，我不记得问我这个问题的人是谁，但我记得我当时回答了

> 哦，我喜欢这个问题！

类似的话。

我喜欢这个问题，因为 Python 3 中的 range 对象（Python 2 中的 xrange）确实是惰性对象，**但 range 对象并不是迭代器**，很多人容易将二者混淆。

在过去的一年里，我听到许多 Python 初学者、资深 Python 的程序员，甚至一些 Python 教育者错误地将 Python 3 中的 range 对象称为迭代器。这个区别确实让很多人感到困惑。

## 混淆情有可原

当人们讨论 Python 中的迭代器和可迭代对象时，可能会听到有人重复这个观点：range 是一个迭代器。

这个错误看起来无关紧要，但我认为实际上它很关键。如果你认为 range 对象是迭代器，那么你对 Python 中迭代器的理解可能还不够深入。虽然 range 和迭代器都带有"惰性"特征，但它们的惰性实现机制却有着本质区别。

本文将解释迭代器如何工作，range 如何工作，以及这两种“惰性可迭代对象”的惰性有何不同。

但首先，我希望你能理解以下内容，并不要用这些信息去责怪任何人，无论是新手还是有经验的 Python 程序员。很多人已经使用 Python 很多年了，并且在完全理解我接下来要解释的这个区别之前，依然能非常愉快地编写代码。你也可以编写成千上万行 Python 代码，而不需要对迭代器如何工作有非常清晰的理解。

## 什么是迭代器？

在 Python 中，可迭代对象是任何可以进行迭代的对象，而迭代器是执行实际迭代操作的对象。

- 可迭代对象能够被迭代。
- 迭代器是执行迭代的“代理”。

你可以通过 `iter()` 函数从任何可迭代对象中获取迭代器：

```python
>>> iter([1, 2])
<list_iterator object at 0x7f043a081da0>
>>> iter('hello')
<str_iterator object at 0x7f043a081dd8>
```

一旦你得到一个迭代器，你可以做的唯一操作就是使用 `next` 获取下一个元素：

```python
>>> my_iterator = iter([1, 2])
>>> next(my_iterator)
1
>>> next(my_iterator)
2
```

如果你请求下一个元素，但没有更多元素了，就会得到一个 `StopIteration` 异常：

```python
>>> next(my_iterator)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

有趣但略微让人困惑的是，所有迭代器也是可迭代的。也就是说，你可以从迭代器中再次获取迭代器（它会返回自己）。因此，你也可以迭代一个迭代器：

```python
>>> my_iterator = iter([1, 2])
>>> [x**2 for x in my_iterator]
[1, 4]
```

需要特别注意的是，迭代器是有状态的。这意味着，一旦你从迭代器中消耗了一个元素，它就被移除了。因此，尝试再次循环迭代一个迭代器时，它会为空：

```python
>>> my_iterator = iter([1, 2])
>>> [x**2 for x in my_iterator]
[1, 4]
>>> [x**2 for x in my_iterator]
[]
```

在 Python 3 中，`enumerate`、`zip`、`reversed` 以及其他一些内建函数返回的就是迭代器：

```python
>>> enumerate([1, 2, 3])
<enumerate object at 0x7f04384ff678>
>>> zip([1, 2], [3, 4])
<zip object at 0x7f043a085cc8>
>>> reversed([1, 2, 3])
<list_reverseiterator object at 0x7f043a081f28>
```

生成器（无论是来自生成器函数还是生成器表达式）是创建自己迭代器的简单方法之一：

```python
>>> numbers = [1, 2, 3, 4, 5]
>>> squares = (n**2 for n in numbers)
>>> squares
<generator object <genexpr> at 0x7f043a0832b0>
```

我常说，**迭代器是“惰性单次可迭代对象”**。它们是“惰性”的，因为它们仅在你遍历它们时计算元素。而它们是“单次”的，因为一旦你从迭代器中“消费”了一个元素，它就永远消失了。通常，迭代器完全消耗完毕时，我们称它们为“耗尽”。

这就是迭代器的简要概述。如果你之前没有接触过迭代器，我建议你进一步复习它们。我曾写过一篇关于迭代器的文章，也曾在 Loop Better 演讲中深入探讨过迭代器的相关内容。

## range 有什么不同？

好，现在我们已经回顾了迭代器，接下来我们来谈谈 range。

Python 3 中的 range 对象（Python 2 中的 xrange）可以像其他可迭代对象一样进行迭代：

```python
>>> for n in range(3):
...     print(n)
...
0
1
2
```

因为 range 是一个可迭代对象，我们可以从中获取一个迭代器：

```python
>>> iter(range(3))
<range_iterator object at 0x7f043a0a7f90>
```

但 range 对象本身并不是迭代器。**我们不能直接对 range 对象调用** `next`：

```python
>>> next(range(3))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'range' object is not an iterator
```

而且与迭代器不同，我们可以多次循环迭代一个 range 对象，而不会消耗它：

```python
>>> numbers = range(3)
>>> tuple(numbers)
(0, 1, 2)
>>> tuple(numbers)
(0, 1, 2)
```

如果我们对一个迭代器这么做，它在第二次迭代时就会变为空：

```python
>>> numbers = iter(range(3))
>>> tuple(numbers)
(0, 1, 2)
>>> tuple(numbers)
()
```

与 `zip`、`enumerate` 或生成器对象不同，range 对象并不是迭代器。

## 那么 range 到底是什么？

range 对象是“惰性”的，因为它在创建时并不会生成所有的数字。相反，它在我们迭代时动态地生成这些数字。

这里是一个 range 对象和一个生成器（迭代器的一种类型）的对比：

```python
>>> numbers = range(1_000_000)
>>> squares = (n**2 for n in numbers)
```

与迭代器不同，range 对象有一个长度：

```python
>>> len(numbers)
1000000
>>> len(squares)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'generator' has no len()
```

并且可以进行索引操作：

```python
>>> numbers[-2]
999998
>>> squares[-2]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'generator' object is not subscriptable
```

与迭代器不同，你可以在不改变其状态的情况下询问 range 对象是否包含某个元素：

```python
>>> 0 in numbers
True
>>> 0 in numbers
True
>>> 0 in squares
True
>>> 0 in squares
False
```

如果你要描述 range 对象，可以称它们为“惰性序列”。它们是序列（像列表、元组和字符串），但它们并不在内存中保存实际的数据，而是通过计算来回答相关问题。

```python
>>> from collections.abc import Sequence
>>> isinstance([1, 2], Sequence)
True
>>> isinstance('hello', Sequence)
True
>>> isinstance(range(3), Sequence)
True
```

## 为什么这种区别很重要？

看起来我的观点 range 不是迭代器 似乎是有点吹毛求疵，但我真的认为这很重要。

如果我告诉你某个对象是迭代器，你会知道，当你对它调用 `iter()` 时，你总是会得到同一个对象（按定义）：

```python
>>> iter(my_iterator) is my_iterator
True
```

你也能确信你可以调用 `next()`，因为所有迭代器都可以调用 `next()`：

```python
>>> next(my_iterator)
4
>>> next(my_iterator)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

并且你会知道，当你遍历一个迭代器时，元素会被消耗掉。有时候，这个特性对于以特定方式处理迭代器非常有用：

```python
>>> my_iterator = iter([1, 2, 3, 4])
>>> list(zip(my_iterator, my_iterator))
[(1, 2), (3, 4)]
```

所以，虽然“惰性可迭代对象”和“迭代器”之间的区别看起来很微妙，但这些术语确实意味着不同的东西。虽然“惰性可迭代对象”是一个很通用的术语，但“迭代器”这个词意味着具有一组非常具体行为的对象。

## 当有疑问的时候，请使用“可迭代对象”或“惰性可迭代对象”

如果你知道某个对象可以被循环迭代，它就是一个可迭代对象。

如果你知道你循环迭代时它会计算元素，那么它就是一个惰性可迭代对象。

如果你知道你可以把某个对象传递给 `next()` 函数，它就是一个迭代器（而迭代器是最常见的惰性可迭代对象）。

**如果你可以多次循环某个对象而不会“耗尽”它，它就不是迭代器。如果你不能把某个对象传递给 **`**next()**`**，它就不是迭代器。**

Python 3 中的 range 对象不是一个迭代器。如果你在讲解 range 对象时，请不要使用“迭代器”这个词。这样做会引起混淆，甚至可能导致别人错误地使用“迭代器”这个词。

另一方面，如果你看到别人错误地使用了“迭代器”这个词，不要生气。如果错误看起来很重要，你可以指出，但要记住，很多长期使用 Python 的程序员甚至经验丰富的 Python 教育者也会错误地称 range 对象为迭代器。**语言是复杂的，虽然词语很重要，但也难以完全规范。**

感谢你参与我这篇关于 range 和迭代器的小冒险！
