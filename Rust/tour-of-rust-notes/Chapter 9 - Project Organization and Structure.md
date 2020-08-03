#2020-07-24

# 写在前面
这是我看 https://tourofrust.com/ 的学习笔记。不一定对，请自行辨别。

# 106 crate
- 每个 program 或者 library 叫做 crate
- 每个 crate 由几个 module 组成
- 每个 crate 有 root 的 module
- module 可以包含 module

# 107 program
program 的 root module 必须写在 `main.rs` 里

# 108 library
library 的 root module 必须写在 `lib.rs` 里

# 109 引用别的 module 和 crate
- 全路径
- `use aa::bb::cc`
- https://crates.io

# 110 引用同个 module 里的多个 item
`use std::f64::consts::{PI,TAU}`

# 111 Module
要创建一个 module foo 
- 要么一个文件 `foo.rs`
- 要么一个文件夹 `foo`，里面有一个文件 `mod.rs`

# 112 Module Hierarchy
介绍了一个 `mod foo;` 的语法。见[[7. Project, Crate, Module#7 5 把 module 拆成不同文件]]

# 113 Inline Sub-Module
举了一个 unit test 的例子

``` Rust
#[cfg(test)]       // <--- 不在 test moode 的时候，这个 mod 不会被编译
mod tests {
    use super::*;   // <--- 需要显式的 use 一下

    ... tests go here ...
}
```
#TODO ： 以上例子哪里体现 inline ?

# 114 一些 use 时的特定关键字
- `use crate::*` 从本 crate 的 root module 开始
- `use super::*` 从上一级 module 开始
- `use self::*` 从当前 module 开始

# 115 export
- 所有成员默认只在 module 里可见
	- Sub-Module 也不可见
- `pub` [[Chapter 7 - Object Oriented Programming#77 field 和 method 的可见性]]
- 如果要把可见性暴露出 crate 外，需要在 **root** module 里 `pub` 之

# 116
struct 的 field 也是要用 `pub` 暴露出 Module 外。

# 117 Prelude
`std::prelude::*`的东西默认会被 Rust 包含。所以不用在自己  use 了。
- `Vec`就是其中之意
- `Box` 也是

# 118 建议自己也把常用的放到一个模块里
- 仿照 `std::prelude::*`
- 但是自己的 `mylib::prelude::*` **不会**被自动包含。只是一个保持和 std 长相相似的手段而已。

# 119 下一步链接
https://rust-lang.github.io/api-guidelines/

# 120 deep dive link
https://doc.rust-lang.org/stable/book/