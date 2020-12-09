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

这段代码用  `gcc` 编译成汇编，有这么一段：

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
    movq    %rax, %rdi     ;; <--- 现在 rdi 里的值是 pa 的值
    call    _Z3fooP1A
    ...

_Z3fooP1A:
.LFB1523:
    ...
    movq    %rdi, -8(%rbp)
    movq    -8(%rbp), %rax  ;; <--- 现在 rax 里的值是 rdi 里的值了，也就是 main 里 pa 的值

    testq    %rax, %rax
    je    .L7

    movq    (%rax), %rax    ;; <--- (%rax) 意思是把 rax 上的值当作指针，把其所指向内存里的内容拿出来；
                            ;; <--- 由于指令是 movq，所以要拿8个 byte；
                            ;; <--- 结合"rax 里现在是 pa 的值"，所以 (%rax) 就是 *pa 开始的8个 byte。
                            ;; <--- 这8个 byte 会被放到后面那个寄存器里，也就是 rax。
                            ;; <--- 所以执行完后，rax 里就是 *pa 的前8个 byte 的内容了。

    movq    -8(%rax), %rax  ;; <--- -8(%rax) 意思是把 rax 上的值减去8得到的新值当作指针，把其所指向内存里的内容拿出来；
                            ;; <--- 由于指令是 movq，所以要拿8个 byte；
                            ;; <--- 结合"rax 里现在是 *pa 的前8个 byte 的内容"；
                            ;; <--- 综合起来就是，*pa 的前8个 byte 本身也是一个指针，将其指向的内存位置，
                            ;; <--- 再之前的那8个 byte 拿出来，放到 rax 里。
    jmp    .L9

.L7:
    call    __cxa_bad_typeid@PLT

.L9:
    movq    %rax, %rdi      ;; <--- 结合下面可知，rax 里面应该是 type_info 的 this 指针
    call    _ZNKSt9type_info4nameEv

 ```



# 问：`*pa` 不是 `B` 的实例吗？为什么汇编里把它当作指针？

因为实际上`B`实例里的第一个字段是虚表的地址；这个地址指向的就是虚表了。



# 问：虚表之前那8个 byte 也是一个指针？

就是 `type_info` 的指针



# 总图

![image-20201209204516794](%E4%B8%80%E6%AE%B5%20typeid%20%E7%9A%84%E6%B1%87%E7%BC%96.assets/image-20201209204516794.png)