#2020-08

# 代码
``` C++
#include <stdio.h>
#include <vector>

struct A {
    int *p;
    A() {
        p = (int *)malloc(sizeof(int));
    }
    A(const A&r) {
        p = (int *)malloc(sizeof(int));
    }
    ~A() {
        if (p){
            free(p);
            p = 0;
        }
    }
};

int main() {
    std::vector<A> a(1);
    const A &x = a[0];

    const int cap = a.capacity();
    for (int i = 0; i < cap; i++) 
        a.push_back(A());

    printf("%d", x.p[0]); 

    return 0;
}
```

# 运行结果
崩溃

# 原因
当`vector`元素个数等于`capacity`时，再`push_back`，就会引发内存扩容。从而`x`所引用当`A`会被析构。所以 `x.p` 就是空指针。所以`x.p[0]`就崩掉里。

# 参考Rust
![[08. Common Collections#上面的例子里，对第一个元素的 borrow 为什么要在意末尾元素的添加？]]