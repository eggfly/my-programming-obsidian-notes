# 问题
实际项目上遇到一个问题，可以简化成这个问题：
```C++
// a.h
inline int aaa() {
#ifdef BBB
    return 6;
#else
    return 7;
#endif
}
```

``` C++
// m.cpp
int bbb();
int ccc();

int main() {
    return bbb() + ccc();
}
```

```C++
// b.cpp
#define BBB
#include "a.h"

int bbb() { return aaa(); }
```

```C++
// c.cpp
#include "a.h"

int ccc() { return aaa(); }
```

在不同的优化配置下结果如下：
```Console
$ g++ -O0 *.cpp && ./a.out
bbb=6
ccc=6

$ g++ -O1 *.cpp && ./a.out
bbb=6
ccc=6

$ g++ -O2 *.cpp && ./a.out
bbb=6
ccc=7

$ g++ -O3 *.cpp && ./a.out
bbb=6
ccc=7
```

为什么在 `-O0` 和 `-O1` 时 `ccc` 的值是7？

# 答案

1. `-O0` 和 `-O1` 时 `inline` 函数不会被真的 `inline`，而是编译成函数，只不过这个函数是 `weak symbol`。
2. 如此，在 `b.cpp` 编译结果里有一个 `aaa` 的`weak symbol`，`c.cpp`编译结果里也有一个`aaa`的 `weak symbol`，而且这两个 `weak symbol` 是不同实现的。
3. 在链接的时候，`aaa`只有两个`weak symbol`而没有 `strong symbol`，所以只有一个会被链接进去，其他的都扔掉 ---- 至于是哪个会被链接，这个好像没有具体的规定？
4. 这就解释了为什 `-O0` 和 `-O1` 时 `bbb` 和 `ccc` 的值是一样的 ---- 他们都用的同一个实现。
