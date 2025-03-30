---
title: "Rust 所有权与引用"
date: 2025-03-30T10:51:39+08:00
lastmod: 2025-03-30T10:51:39+08:00
author: ["熊大如如"]
summary: "rust 所有权和引用"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202503301055444.jpg" # 文章的图片
---

## Rust 所有权原则

1. Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
2. 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
3. 当所有者（变量）离开作用域范围时，这个值将被丢弃

### 变量所有权

在 Rust 中，默认情况下赋值操作会转移所有权，而不是创建副本。例如

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // s1 的所有权转移给 s2

    // println!("{}", s1); // ❌ 编译错误，s1 不再有效
}
```

- `s1` 原本是 `"hello"` 的所有者
- `s1` 赋值给 `s2` 后，所有权转移，`s1` 失效，不能再被使用

对于基本数据类型（如整数、浮点数、布尔值、字符等），Rust 采用 Copy 一个值，不会发生所有权转移

```rust
fn main() {
    let x = 5;
    let y = x; // x 仍然可用，因为 i32 是 Copy 类型

    println!("x = {}, y = {}", x, y); // ✅ 允许
}
```

### 变量作用范围

```rust
fn main() {
    {
        let s = "ss";
    }
    println!("s 的数值是 {}", s);
}
```

`cargo check`报错

```rust
error[E0425]: cannot find value `s` in this scope
 --> src\main.rs:5:27
  |
5 |     println!("s 的数值是 {}", s);
  |                               ^
  |
help: the binding `s` is available in a different scope in the same function
 --> src\main.rs:3:13
  |
3 |         let s = "ss";
  |             ^

For more information about this error, try `rustc --explain E0425`.
```

- 不能在这个作用域中发现 `s`

简而言之，`s` 从创建开始就有效，然后有效期持续到它离开作用域为止

### 深拷贝

首先，Rust 永远也不会自动创建数据的 深拷贝 因此，任何自动的复制都不是深拷贝，可以被认为对运行时性能影响较小

- 手动深拷贝

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

## Rust 的引用

Rust 的 引用（Reference） 是一种不拥有数据所有权的方式，它允许在不复制数据的情况下访问数据，同时确保 内存安全 和 数据一致性

引用的本质是指向某个值的地址，但它不会夺取该值的所有权，而只是借用（borrow）它

### 示例

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // 传递 s1 的引用
    println!("The length of '{}' is {}", s1, len); // ✅ s1 仍然可用
}

fn calculate_length(s: &String) -> usize {
    s.len() // 只读访问 s
}
```

- `&s1` 代表对 `s1` 的引用，但 `s1` 仍然是原来的所有者
- `calculate_length` 只是借用 `s`，不会修改它，也不会夺走所有权
- `s1` 在 `main` 里仍然有效

### 可变引用

如果想修改引用的数据，需要使用可变引用（`&mut T`）

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s); // 可变借用 s
    println!("{}", s); // ✅ "hello, world!"
}

fn change(s: &mut String) {
    s.push_str(", world!"); // 修改 s
}
```

**可变引用的规则**

1. 在同一时间，只能有一个可变引用（&mut T）
2. 不能同时拥有可变引用和不可变引用（&T）

### 不可变引用

```rust
fn main() {
    let s = String::from("hello");

    let r1 = &s; // 允许
    let r2 = &s; // 允许
    println!("{} and {}", r1, r2); // ✅ 允许多个不可变引用
}
```

### 可变引用和不可变引用不能同时存在

Rust 防止数据竞争，所以：

- 如果存在可变引用（&mut T），就不能有不可变引用（&T）
- 如果存在不可变引用（&T），就不能创建可变引用（&mut T）

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    let r3 = &mut s; // ❌ 编译错误：r1, r2 仍然有效，不能创建可变引用

    println!("{}, {}", r1, r2);
}
```

### 悬垂引用

```rust
fn dangle() -> &String { // ❌ 编译错误
    let s = String::from("hello");
    &s // ❌ 返回局部变量的引用
} // s 被释放，引用失效
```

- 这里引用值会释放掉，从而造成错误
- Rust 检查器会报错

其中一个很好的解决方法是直接返回 String

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s // ✅ 所有权转移，数据不会被释放
}
```
