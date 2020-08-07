#2020-08

# Codes
``` C
int foo(int n) {
    int a[n];
    int b;
    return b + a[0] + a[1];
}
```

https://godbolt.org/ 上 "x86-64 gcc 10.2" 看汇编：

```
foo:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 48
    mov     DWORD PTR [rbp-36], edi
    mov     rax, rsp
    mov     rcx, rax
    mov     eax, DWORD PTR [rbp-36]
    movsx   rdx, eax
    sub     rdx, 1
    mov     QWORD PTR [rbp-8], rdx
    movsx   rdx, eax
    mov     r10, rdx
    mov     r11d, 0
    movsx   rdx, eax
    mov     r8, rdx
    mov     r9d, 0
    cdqe
    lea     rdx, [0+rax*4]
    mov     eax, 16
    sub     rax, 1
    add     rax, rdx
    mov     esi, 16
    mov     edx, 0
    div     rsi
    imul    rax, rax, 16
    sub     rsp, rax
    mov     rax, rsp
    add     rax, 3
    shr     rax, 2
    sal     rax, 2
    mov     QWORD PTR [rbp-16], rax
    mov     rax, QWORD PTR [rbp-16]
    mov     edx, DWORD PTR [rax]
    mov     eax, DWORD PTR [rbp-20]
    add     edx, eax
    mov     rax, QWORD PTR [rbp-16]
    mov     eax, DWORD PTR [rax+4]
    add     eax, edx
    mov     rsp, rcx
    leave
    ret
```

#TODO 