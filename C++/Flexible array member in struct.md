#2020-08

# Codes

``` C
int foo() {
    struct A {
        int yyy;
        int xxx[];  // <--- 没有写长度
    };
    return sizeof(struct A);
}

int bar() {
    struct A {
        int yyy;
        int xxx[0]; // <--- 长度是0
    };
    return sizeof(struct A);
}

int qux() {
    struct A {
        int yyy;
        int xxx[1]; // <--- 长度是1
    };
    return sizeof(struct A);
}

```

在 https://godbolt.org/ 上看汇编，选 "x86-64 gcc 10.2"，得到结果如下：
``` 
foo:
        push    rbp
        mov     rbp, rsp
        mov     eax, 4    // <--- 说明 int xxx[] 不占长度
        pop     rbp
        ret
bar:
        push    rbp
        mov     rbp, rsp
        mov     eax, 4    // <--- 说明 int xxx[0] 不占长度
        pop     rbp
        ret
qux:
        push    rbp
        mov     rbp, rsp
        mov     eax, 8    // <--- 说明 int xxx[1] 占1个int的长度
        pop     rbp
        ret
```

# 理论
- 这个`int xxx[]`叫 `flexible array members`，是 `C99` 的东西。
- 注意`int xxx[0]` 不是 `flexible array member`。。。

# 限制
- 那个 `flexible array member` 必须是 `struct` 的最后一个成员。
``` C
struct A {
    int xxx[]; // <--- 这个通不过编译。因为不是最后一个成员。
    int a;
};

struct B {
    int xxx[0]; // <--- 但是这个能通过编译。它不是 flexible array member。
    int a;
};
```

- `struct` 里必须要除了 `flexiable array member` 外的其他东西。
``` C
struct A {
    int xxx[]; // <--- 这个通不过编译。因为没有 named member
};

struct B {
    int xxx[0]; // <--- 但是这个能通过编译。它不是 flexible array member。
};
```

# 更多讨论
https://stackoverflow.com/questions/246977/is-using-flexible-array-members-in-c-bad-practice