#2020-07

## 问题
刚刚在 https://www.bfilipek.com/2019/02/lambdas-story-part1.html 看一个东西，发现一个`capture class member`的例子。我构造来如下代码。问题是：那个`lambda capture`的是那个类的`this`指针，还是那个`a`的`int`？

```
struct A
{
    int b;
    int a;
    int foo() {
        auto fn = [=]{return a;};
        return fn();
    }
};

int foo() { A().foo(); }
```

## 答案
是 `this` 指针。通过在 https://godbolt.org/ 上编译看汇编，有如下结果

```
A::foo()::{lambda()#1}::operator()() const:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi  // <--- rbp-8 现在是 old_rbp-8 的地址
        mov     rax, QWORD PTR [rbp-8]  // <--- rax 现在是 old_rbp-8 的地址
        mov     rax, QWORD PTR [rax]    // <--- rax 现在是 old_rbp-8 的内容，也就是 A 的 this指针值
        mov     eax, DWORD PTR [rax+4]  // <--- eax 现在是 A::b 的值
        pop     rbp
        ret
A::foo():
        push    rbp
        mov     rbp, rsp
        sub     rsp, 32
        mov     QWORD PTR [rbp-24], rdi
        mov     rax, QWORD PTR [rbp-24] // <--- rax 现在是 A 的 this 指针值
        mov     QWORD PTR [rbp-8], rax  // <--- rbp-8 现在是 A 的 this 指针值
        lea     rax, [rbp-8]
        mov     rdi, rax                // <--- rdi 现在是 rbp-8 的地址
        call    A::foo()::{lambda()#1}::operator()() const
        leave
        ret
foo():
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-8], 0
        mov     DWORD PTR [rbp-4], 0
        lea     rax, [rbp-8] // <--- rax 现在是 A 的 this 指针值
        mov     rdi, rax     // <--- rdi 现在是 A 的 this 指针值
        call    A::foo()
        nop
        leave
        ret
```