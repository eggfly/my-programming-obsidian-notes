#2020-07

# 写在前面
这是我看 https://tourofrust.com/ 的学习笔记。不一定对，请自行辨别。

# 74 什么是OOP
- 封装
- 抽象
- 多态
- 继承

# 75 Rust 不是 OOP
因为：
- 不能继承 field
- 不能继承 method

# 76 对 method 的封装
- 不改变 object 的 method 用 &self

``` Rust
struct A {}

impl A {
    fn foo(&self) {
        println!("A::foo()")
    }
}

fn main() {
    let a = A{};
    a.foo();
}
```

- 改变 object 的 method 用 &mut self
``` Rust
struct A {
    a: i32,
}

impl A {
    fn foo(&mut self) {
        self.a = 4;
        println!("a={}", self.a);
    }
}

fn main() {
    let mut a = A { a: 3}; // <---  注意 a 也必须是 mut 的
    a.foo();
}
```

# 77 field 和 method 的可见性
- 默认 field 和 method 都是 module 可见的
- 可以用 `pub` 关键字

``` Rust
struct A {
    pub a : i32,
}

impl A {
    pub fn foo(&self) {
        println!("A::foo()");
    }
}

fn main() {
    let a = A{ a: 3};
    
    println!("{}", a.a);
    a.foo();
}
```

## 78 用 Traits 来实现多态
- trait 类似于 Java 里的 interface 或者 abstract class
- 用 impl MyTrait for MyStruct 的语法

``` Rust
struct A {}
struct B {}

trait C {
    fn foo(&self);
}

impl C for A {
    fn foo(&self) {
        println!("A::foo()");
    }
}

impl C for B {
    fn foo(&self) {
        println!("B::foo()");
    }
}

fn main() {
    let a = A{};
    let b = B{};
    a.foo();
    b.foo();
}
```

## 79 Traits 也可以有默认实现
``` Rust
struct A{
    a : i32,
}

trait B {
    fn foo(&self) {
        println!("B::foo()");
    }
}

impl B for A {}

fn main() {
    let a = A{ a: 3 };
    a.foo();
}
```
注意上面例子里的 b::foo() 并不能访问 A::a.

## 80 Trait 是可以继承的
``` Rust
struct A {}

trait B {
    fn foo(&self) {
        println!("B::foo");
    }
}

trait C : B {
    fn bar(&self) {
        self.foo();
        println!("C::bar");
    }
}

impl B for A {}
impl C for A {}

fn main() {
    let a = A{};
    a.bar();
}
```
注意 `C : B`是必要的，否则会有如下编译错误
``` Rust
struct A {}

trait B {
    fn foo(&self) {
        println!("B::foo");
    }
}

trait C {
    fn bar(&self) {
        self.foo();
        println!("C::bar");
    }
}

impl B for A {}
impl C for A {}

fn main() {
    let a = A{};
    a.bar();
}

=============
error[E0599]: no method named `foo` found for reference `&Self` in the current scope
  --> src/main.rs:11:14
   |
11 |         self.foo();
   |              ^^^ method not found in `&Self`
   |
   = help: items from traits can only be used if the type parameter is bounded by the trait
help: the following trait defines an item `foo`, perhaps you need to add a supertrait for it:
   |
9  | trait C: B {
   |        ^^^
```

### 一个不是 trait 继承的例子
下面这个例子里，C并没有继承B，但是在A::bar里面是可以调用的self.foo()的
``` Rust
struct A {}

trait B {
    fn foo(&self);
}

trait C {
    fn bar(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

impl C for A {
    fn bar(&self) {
        self.foo();
        println!("C::bar");
    }
}

fn main() {
    let a = A{};
    a.bar();
}
```

## 81 static dispatch 和 dynamic dispatch
- 如果编译器 compile time 能确定 struct 的类型，就会使用 static dispatch
- 如果编译器 run time 才能确定 struct 的类型，就会使用 dynamic dispatch
- dynamic dispatch 会慢一点

``` Rust
struct A{}

trait B {
    fn foo(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

fn foo_a(a: &A) {
    a.foo(); // <--- static dispatch
}

fn foo_b(b: &dyn B) { // <--- 注意那个 dyn 关键字
    b.foo(); // <--- dynamic dispatch
}

fn main() {
    let a = A{};
    foo_a(&a);
    foo_b(&a);
}
```

对于 dynamic dispatch，鼓励在代码里加上 `dyn` 字眼。如果不加，其实也可以执行；但是会有编译错误：
``` Rust
struct A{}

trait B {
    fn foo(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

fn foo_b(b: & B) { // <--- 注意没有 dyn 关键字
    b.foo(); // <--- dynamic dispatch
}

fn main() {
    let a = A{};
    foo_b(&a);
}

=============
warning: trait objects without an explicit `dyn` are deprecated
  --> src/main.rs:13:15
   |
13 | fn foo_b(b: & B) { // <--- 注意那个 dyn 关键字
   |               ^ help: use `dyn`: `dyn B`
   |
   = note: `#[warn(bare_trait_objects)]` on by default

warning: 1 warning emitted
```

## 82 Trait Object
在上面 [[Chapter 7 - Object Oriented Programming#81 static dispatch 和 dynamic dispatch]] 的例子里，`foo_b(b: &dyn B)` 里的 `&dyn B` 叫做 trait object。就是C++里全是虚函数的基类。

## 83 Unsized data
- 在一个 struct 里存储另一个 trait 的时候，可能会增加原有 struct 的内存尺寸。如果不想增加，则要用 unsized value 等方式引入另一个trait。
- Rust里主要有两种 unsized value
	- generics
		- [[Chapter 7 - Object Oriented Programming#84 Generic Function]]
		- [[Chapter 7 - Object Oriented Programming#87 Generic Struct]]
	- 只存一个指向那个trait的指针。
		- 严格来说，这个增加了原有 struct 的尺寸，不过由于只增加了一个指针，大部分情况下可以接受。
		- 这个是用 `Box` 来实现的。见 [[Chapter 7 - Object Oriented Programming#86 Box]].

## 84 Generic Function
- 语法是 `fn foo<A>(a: A) where A: B {}`。B必须是 trait。
- compile time 能确定类型。所以是 static dispatch。
``` Rust
struct A {}

trait B {
    fn foo(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

fn foo<T> (a: &T) where T:B { // <--- B必须是 Trait
    a.foo();
}

fn main() {
    let a = A{};
    foo(&a);
}
```

## 85 Generic Function 的一个语法糖
```
struct A {}

trait B {
    fn foo(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

fn foo (a: & impl B) { // <--- 注意 impl B
    a.foo();
}

fn main() {
    let a = A{};
    foo(&a);
}
```

## 86 Box
- `Box` 用于把数据从栈上移到堆上
- `Box` 基本指包含里一个指针，指向堆上的一个内存
- `Box` 是 **智能指针**
- `Box::new(A {...})`
- `Box::new(&A)`

``` Rust
struct A {
    a: i32,
}

impl A {
    fn foo(&self) {
        println!("{}", self.a);
    }
}

fn main() {
    let a = A { a : 3 };
    let b = A { a : 4 };
    
    let v = vec![Box::new(&a),
                 Box::new(&b),
                 Box::new(&A { a : 5 })];
    
    for x in v.iter() {
        x.foo();
    }
}
```

## 87 Generic Struct
``` Rust
struct A {}

trait B{
    fn foo(&self);
}

impl B for A {
    fn foo(&self) {
        println!("B::foo");
    }
}

struct C<T> where T:B { // <--- 看这里
    t:T,
}

impl<T> C<T> where T:B { // <--- 看这里
    fn foo(&self) {
        self.t.foo();
    }
}

fn main() {
    let c = C { t : A{} };
    c.foo();
}
```

## 88 总结
略过