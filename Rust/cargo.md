#2020-07
#2020-08

# cargo new
空文件下默认会添加`.gitignore`

# cargo init
添加 toml

# TOML
TOML - Tom's Obvious, Minimal Lanaguage

https://doc.rust-lang.org/cargo/reference/manifest.html

``` toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"
[dependencies]
rand = "0.5.5" // <--- 是 ^0.5.5 的简写，意思是任何兼容 0.5.5 的版本
```

## SemVer
以上 dependency 里的版本号是 SemVer https://semver.org/lang/zh-CN/

# cargo build
默认文件在 `./target/debug/` 文件夹下。

可能会生成 `cargo.lock` 文件。这个是和项目依赖相关的。
![[02. Programming a Guessing Game#cargo lock]]

`cargo build --release`

# cargo run
编译并运行

# cargo check
检查是否能编译，但并不真的编译。

这个比 [[cargo#cargo build]] 快。

# cargo使用国内镜像
创建`~/.cargo/config`，内容如下：
```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```
