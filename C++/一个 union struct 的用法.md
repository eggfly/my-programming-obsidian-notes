# Codes

``` C
#include <stdio.h>

struct A {
    union {
        struct{ int a; int b; };  // <--- 没有定义名字
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

# 笔记
按照常见用法，A的定义会写成
``` C
struct A{
    union{
        struct{ int a; int b; } xx;
        int c[2];
    };
};
```

然后用的时候：
``` C
A a;
printf("%u %d %d\n", sizeof(struct A), a.xx.a, a.xx.b);
```

可是上面的例子里并没有定义这样的xx，可以能用。