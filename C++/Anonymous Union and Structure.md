#2020-08

# Codes

``` C
#include <stdio.h>

struct A {
    union {
        struct { int a; int b; };  // <--- 没有定义名字
        int c[2];
    };
};

int main()
{
    A a;
    a.c[0] = 3;
    a.c[1] = 4;

    printf("%u %d %d\n", sizeof(struct A), a.a, a.b);
}
```

# 运行结果
```
$ gcc a.c && ./a.out
8 3 4
```

# 分析 
按照`C99`写法，A的定义会写成
``` C
struct A {
    union {
        struct{ int a; int b; } xx;
        int c[2];
    } yy;
};
```

然后用的时候：
``` C
A a;
a.yy.c[0] = 3;
a.yy.c[1] = 4;
printf("%u %d %d\n", sizeof(struct A), a.yy.xx.a, a.yy.xx.b);
```

可上面的例子里并没有定义这样的`xx`和`yy`，但也能用。这是**C11**的标准；另外在`C11`之前，`GCC`也有这个扩展。

# 更多例子
下面这个也能用：
``` C
#include <stdio.h>

struct A {
    struct {
        int a;
        int b;
    };
};

int main()
{
    struct A a;
    a.a = 3;
    a.b = 4;

    printf("%lu %d %d\n", sizeof(struct A), a.a, a.b);

    return 0;
}
```

但是下面这个就不能用：
``` C
#include <stdio.h>

struct A {
    struct {
        int a;
        int b;
    };
	int a;     // <--- 编译错误：error: duplicate member ‘a’
};

int main()
{
    struct A a;
    a.a = 3;
    a.b = 4;

    printf("%lu %d %d\n", sizeof(struct A), a.a, a.b);

    return 0;
}
```

# 参考链接
https://www.geeksforgeeks.org/g-fact-38-anonymous-union-and-structure/

https://gcc.gnu.org/onlinedocs/gcc/Unnamed-Fields.html

https://stackoverflow.com/a/8932782