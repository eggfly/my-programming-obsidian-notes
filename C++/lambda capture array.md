#2020-07

## Capture by value on array 
今天我遇到一个 [[`lambda`]] 对数组 [[`capture by value`]] 的问题。简而言之就是，以下内容是 [[`lambda capture array`]] 的。

```
int foo(int num) {
    int a[3] = {1,2,3};
    auto fn = [a]() {return a[0] + a[2];};
    a[0] = 9;
    return fn();
}
```

在 [[https://godbolt.org/]] 上看汇编，是这样的 (注释是我写的，我用 `old_rbp` 来指代`foo(int)` 中的那个 `rbp`， 因为我不知道怎么表示 )：

```
foo(int)::{lambda()#1}::operator()() const:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi     // <---现在 rbp-8 存的是 old_rbp-24 的地址
        mov     rax, QWORD PTR [rbp-8]     // <---现在 rax 存的是 old_rbp-24 的地址
        mov     edx, DWORD PTR [rax]       // <---现在 edx 存的是 old_rbp-24 的内容，也就是1
        mov     rax, QWORD PTR [rbp-8]
        mov     eax, DWORD PTR [rax+8]     // <---现在 edx 存的是 old_rbp-24+8 的内容，也就是 old_rbp-16 的内容，也就是3
        add     eax, edx
        pop     rbp
        ret
foo(int):
        push    rbp
        mov     rbp, rsp
        sub     rsp, 40
        mov     DWORD PTR [rbp-36], edi
        mov     DWORD PTR [rbp-12], 1       // <---这里是a[0]
        mov     DWORD PTR [rbp-8], 2        // <---这里是a[1]
        mov     DWORD PTR [rbp-4], 3        // <---这里是a[2]
        mov     rax, QWORD PTR [rbp-12]
        mov     QWORD PTR [rbp-24], rax     // <---现在 [rbp-24] 是 1
        mov     eax, DWORD PTR [rbp-4]
        mov     DWORD PTR [rbp-16], eax     // <---现在 [rbp-16] 是 3
        mov     DWORD PTR [rbp-12], 9       // <--- a[0] = 9
        lea     rax, [rbp-24]
        mov     rdi, rax                     // rdi 现在存的是 rbp-24 这个地址
        call    foo(int)::{lambda()#1}::operator()() const
        leave
        ret
```

可见，确实是把`a`拷贝一遍再去算的。我之前以为`a`是一个指针，数组内容不会被拷贝，我的理解是错的。

## 变化
但如下代码，则不会拷贝`a`那个数组的内容。变化就是`lambda capture`的时候用了`b=a`.
```
int foo(int num) {
    int a[3] = {1,2,3};
    auto fn = [b=a]() {return b[0] + b[2];};
    a[0] = 9;
    return fn();
}
```
在 [[https://godbolt.org/]] 上看汇编，是这样的 (注释是我写的，我用 `old_rbp` 来指代`foo(int)` 中的那个 `rbp`， 因为我不知道怎么表示 )：
```
foo(int)::{lambda()#1}::operator()() const:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-8], rdi  // <--- rbp-8 存的是 old_rbp-24 这个地址
        mov     rax, QWORD PTR [rbp-8]  // <--- rax 存的是 old_rbp-24 这个地址
        mov     rax, QWORD PTR [rax]    // <--- rax 存的是 old_rbp-12 这个地址
        mov     edx, DWORD PTR [rax]    // <--- edx 存的是 old_rbp-12 的内容，也就是1
        mov     rax, QWORD PTR [rbp-8]  // <--- rax 存的是 old_rbp-24 这个地址
        mov     rax, QWORD PTR [rax]    // <--- rax 存的是 old_rbp-12 这个地址
        add     rax, 8                  // <--- rax 存的是 old_rbp-4 这个地址
        mov     eax, DWORD PTR [rax]    // <--- eax 存的是 old_rbp-4 的内容，也就是3
        add     eax, edx
        pop     rbp
        ret
foo(int):
        push    rbp
        mov     rbp, rsp
        sub     rsp, 40
        mov     DWORD PTR [rbp-36], edi
        mov     DWORD PTR [rbp-12], 1     // <--- a[0]
        mov     DWORD PTR [rbp-8], 2      // <--- a[1]
        mov     DWORD PTR [rbp-4], 3      // <--- a[2]
        lea     rax, [rbp-12]             // <--- rax 存的是 rbp-12 这个地址
        mov     QWORD PTR [rbp-24], rax   // <--- rbp-24 存的是 rbp-12 这个地址
        mov     DWORD PTR [rbp-12], 9
        lea     rax, [rbp-24]             // <--- rax 存的是 rbp-24 这个地址
        mov     rdi, rax                  // <--- rdi 存的是 rbp-24 这个地址
        call    foo(int)::{lambda()#1}::operator()() const
        leave
        ret
```

## codes
以下是代码和运行结果
```
$ cat a.cpp && echo '---' && gcc --std=c++14 a.cpp && ./a.out
#include <stdio.h>

int foo1() {
	int a[3] = {1,2,3};
	auto fn = [a]() {return a[0] + a[2];};
	a[0] = 9;
	return fn();
}

int foo2() {
	int a[3] = {1,2,3};
	auto fn = [b=a]() {return b[0] + b[2];};
	a[0] = 9;
	return fn();
}

int main()
{
	printf("foo1()=%d\n", foo1());
	printf("foo2()=%d\n", foo2());
	return 0;
}
---
foo1()=4
foo2()=12

```