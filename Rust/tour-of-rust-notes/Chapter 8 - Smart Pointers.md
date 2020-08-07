#2020-07

# 写在前面
这是我看 https://tourofrust.com/ 的学习笔记。不一定对，请自行辨别。

# 90 重新看引用
- 引用的内部就是指针
- Rust 会对引用对生命周期有特殊的处理

# 91 Raw Pointer
- 两种 raw pointer
	- `*const T`
	- `*mut T`
- raw pointer 可以和数字互转
- raw pointer 可以在 unsafe 模式下使用
- 和 C 指针的对比：Rust 的 Raw Pointer 在 compile time 下限制更多

``` Rust
fn main() {
    let a = 3;
    println!("{}", &a as *const i32 as usize); // <--- * const i32
}
```

# 92 deference
略过

# 93 `*` 操作符
``` Rust
fn main() {
    let a: i32 = 42;
    let rrra: &&&i32 = &&&a;
    let rra: &&i32 = *rrra;
    let ra: &i32 = **rrra;
    println!("{} {} {}", ***rrra, **rra, *ra);
    println!("{} {} {}", *rrra, *rra, *ra);
}
```

问题：上面 `*rrra` 和 `*rra` 为什么也是输出 42？ #TODO 

# 94 `.` 操作符
``` Rust
struct A {
    a: i32,
}

fn main() {
    let a = A { a : 42 };
    let rrra: &&&A = &&&a;
    let rra: &&A = *rrra;
    let ra: &A = **rrra;
    println!("{} {} {} {}", rrra.a, rra.a, ra.a, a.a);
}
```
编译器自动把 `rrra.a` 变成 `(***rrra).a` 了。

# 95 smart pointer
- 三种 Traits
	- `Deref`
	- `DerefMut`
	- `Drop`
- 重写 `*` 和 `.` 操作符

``` Rust
struct A {}

impl std::ops::Deref for A {
    type Target = i32;
    fn deref(&self) -> &i32 {
        &6
    }
}

fn main() {
    let a = A {};
    println!("{}", *a);
}
```

#TODO : 这一节我没有真懂

# 96 Smart Unsafe Code
- 编译器无法保护 unsafe 代码
- unsafe代码可以操作 raw pointer
- `unsafe {...}`

``` Rust
fn main() {
    let a: u16 = 257;
    let ra = &a as *const u16 as *const i8;
    println!("{}", unsafe { *ra }); // <--- result: 1
}
```

# 97 一些老朋友
- `Vec<T>` 和 `String` 都是 smart pointer
- 用 `alloc` 和 `Layout`

``` Rust
fn main() {
    let layout = std::alloc::Layout::from_size_align(4, 1).unwrap();
    let a = unsafe {
        let p = std::alloc::alloc(layout) as *mut u8;
        
        *p = 1;
        *p.offset(1) = 1;
        
        *(p as *const i32)
    };
    println!("{}", a);
}
```

# 98 堆上的内存
这一节说 `Box`

``` Rust
struct A {}

impl A {
    fn foo(&self) {
        println!("A::foo")
    }
}

fn main() {
    Box::new(A {}).foo();
}
```

# 99 再看 Failable Main
自定义Error的例子
``` Rust
#[derive(Debug)]
struct A {}

impl core::fmt::Display for A {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "A")
    }
}

impl std::error::Error for A {}

fn foo() -> Result<(), Box<dyn std::error::Error>> {
    Err(Box::new(A {}))
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    foo()?;
    Ok(())
}
```

# 100 Referencing Counting
``` Rust
struct A {}

impl A {
    fn foo(&self) {
        println!("A::foo");
    }
}

fn main() {
    let a = std::rc::Rc::new(A {});
    let b = a.clone();
    let c = b.clone();
    
    a.foo();
    b.foo();
    c.foo();
}
```

# 101 `RefCell`
要么只读不能写，要么只有一个写不能读。

``` Rust
struct A {}

impl A {
    fn foo(&mut self) {
        println!("A::foo(&mut)");
    }

    fn bar(& self) {
        println!("A::bar(&)");
    }
}

fn main() {
    let a = std::cell::RefCell::new(A {});

	//注意如果没有下面这对花括号，则再下面的 a.borrow() 会崩溃
    {
        let mut mutra = a.borrow_mut();
        mutra.foo();
    }

    let ra = a.borrow();
    let rb = a.borrow();
    ra.bar();
    rb.bar();
}
```

#TODO `RefCell` 被广泛使用了吗？代价是什么？

# 102 `Mutex`
```
struct A {}

impl A {
    fn foo(&self) {
        println!("A::foo");
    }
}

fn main() {
    let ma = std::sync::Mutex::new(A {});
    let refa = ma.lock().unwrap();
    refa.foo();
}
```

# 103 SmartPointer的组合
略过，因为我还没有掌握那几个数据结构。例子里用的是 `std::rc::Rc<std::cell::RefCell<A>>`