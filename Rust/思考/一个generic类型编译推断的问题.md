#2020-08 

# Rust Codes
如下代码，`a` 能通过编译，是因为编译器可以从 5 和 4 推断出两个数的类型，而且**两个数的类型是一致的**，都是i32，从而 `T` 就是 i32。但是 `b` 通不过编译，因为 5 和 4.0 的类型分别是 i32 和 f64，但是编译器**不会自动**把 i32 提升成 f64。
``` rust
struct A<T> { a: T, b: T }

fn main() {
    let a = A { a: 5, b: 4};
    let b = A { a: 5, b: 4.0 };
}
```
错误信息如下
```
error[E0308]: mismatched types
 --> src/main.rs:5:26
  |
5 |     let b = A { a: 5, b: 4.0 };
  |                          ^^^ expected integer, found floating-point number

error: aborting due to previous error
```

可见rust 是**先从 `a:5` 确定了`A`的`T`参数是 i32，再用它去检查其他的参数**。

# C++ class template codes
``` C++
template <typename T>
struct A{
        T a;
        T b;
};

void foo() {
        auto a = A(3, 4.0);
}      
```
同样编译错误，但是错误信息是：
```
$ g++ a.cpp
a.cpp: In function ‘void foo()’:
a.cpp:8:12: error: missing template arguments before ‘(’ token
    8 |  auto a = A(3, 4.0);
      |            ^
```
需要显式把类型写出来，**没有自动推断**。

# C++ function template codes
``` C++
template <typename T> void foo(T x, T y) {}

void bar() {
        foo(3, 4.0);
}
```
同样编译错误，但是错误信息是：
```
$ g++ a.cpp
a.cpp: In function ‘void bar()’:
a.cpp:4:12: error: no matching function for call to ‘foo(int, double)’
    4 |  foo(3, 4.0);
      |            ^
a.cpp:1:28: note: candidate: ‘template<class T> void foo(T, T)’
    1 | template <typename T> void foo(T x, T y) {}
      |                            ^~~
a.cpp:1:28: note:   template argument deduction/substitution failed:
a.cpp:4:12: note:   deduced conflicting types for parameter ‘T’ (‘int’ and ‘double’)
    4 |  foo(3, 4.0);
      |            ^
```

可见g++编译器的行为是**先确定所有参数的类型，再去看有没有对应函数类型**

