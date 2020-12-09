# 问：解读如下那段汇编

（答案inline了）

``` C++
#include <iostream>
#include <typeinfo>

class A {
  virtual void bar() {}
};

class B : public A {};

void foo(A *pa) { std::cout << typeid(*pa).name() << std::endl; }

int main() {
  A *pa = new B();
  foo(pa);
  delete pa;
  return 0;
}
```

这段代码用  `gcc` 编译成汇编，有这么一段：（为了方便描述，汇编的注释里均用 `pa` 来指代 `main` 函数里那个 `B` 实例的地址，用 `*`来指代一个地址指向的内存）

``` console
$ g++ -v
...
gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)
```

 ``` assembly
$ gcc -S a.cpp && cat a.s
 main:
    ...
    ;; foo(pa)
    movq    %rax, %rdi     ;; <--- 执行后 rdi 里的值是 pa 的值
    call    _Z3fooP1A
    ...

_Z3fooP1A:
.LFB1523:
    ...
    movq    %rdi, -8(%rbp)
    movq    -8(%rbp), %rax  ;; <--- 执行后 rax 里的值是 rdi 里的值了，也就是 main 里 pa 的值

    testq    %rax, %rax
    je    .L7

    movq    (%rax), %rax    ;; <--- 执行前 rax 里是 pa 的值
                            ;; <--- 执行后 rax 里是 *pa 的前八个 byte 的内容

    movq    -8(%rax), %rax  ;; <--- 执行前 rax 里是 *pa 的前八个 byte 的内容
                            ;; <--- 执行后 rax 里是 **pa 之前（exclusive）八个 byte 的内容
    jmp    .L9

.L7:
    call    __cxa_bad_typeid@PLT

.L9:
    movq    %rax, %rdi      ;; <--- 结合下面可知，rax 里面应该是 type_info 的 this 指针
    call    _ZNKSt9type_info4nameEv

 ```



# 问：`*pa` 不是 `B` 的实例吗？为什么汇编里把它当作指针？

值这一段：

``` assembly
movq    (%rax), %rax
movq    -8(%rax), %rax
```

因为实际上`B`实例里的第一个字段是虚表的地址；这个地址指向的就是虚表。



# 问：虚表之前那8个 byte 也是一个指针？

就是 `type_info` 的指针



# 总图

![image-20201209204516794](%E4%B8%80%E6%AE%B5%20typeid%20%E7%9A%84%E6%B1%87%E7%BC%96.assets/image-20201209204516794.png)



# 问：如果 A 里没有虚函数？

``` C++
#include <iostream>
#include <typeinfo>

class A {};

class B : public A {};

void foo(A *pa) { std::cout << typeid(*pa).name() << std::endl; }

int main() {
  A *pa = new B();
  foo(pa);
  delete pa;
  return 0;
}
```

则 `foo` 的汇编里，直接获得了`A`的 `typeinfo` 的地址了。注意是 `A` 的，不是 `B` 的。

``` assembly
_Z3fooP1A:
.LFB1522:
        .cfi_startproc
        endbr64
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $16, %rsp
        movq    %rdi, -8(%rbp)
        leaq    _ZTI1A(%rip), %rdi       ;; <--- 这一行直接获得了 A 的 typeinfo 的内存地址
        call    _ZNKSt9type_info4nameEv
        ...
```

