#2020-08

# 问题
今天遇到一个问题。经过一番探索，可以把问题简化为：实现一个函数把 `string` 解析成 `double`，且能通过这个测试用例：`assert 0.05 + 3 * atof("0.04") < 0.17000000000000001;`

``` C
#include <stdio.h>

double my_atof(const char *str) {
	// TODO
}

int main() {
	bool ok = (0.05 + 3 * my_atof("0.04") < 0.17000000000000001);
	printf(ok ? "test pass\n" : "test fail\n");
	return 0;
}
```

# 第一直觉
第一直觉是写一个函数，通过 `ab.cd = a*10 + b*1 + c*0.1 + d*0.1*0.1` 这样的公式来算。可是会失败：
```
$ cat b.cpp && echo '-----' && g++ b.cpp -o b.out && ./b.out
#include <stdio.h>
#include <stdlib.h>

double my_atof(const char *str) {
    return 0.1 * 0.1 * 4;
}

int main() {
    bool ok = (0.05 + 3 * my_atof("0.04") < 0.17000000000000001);
    printf(ok ? "test pass\n" : "test fail\n");
    return 0;
}
-----
test fail
```

# 分析
写了一段函数看各个数的二进制表示：
```
$ cat c.cpp && echo '-----' && g++ c.cpp -o c.out && ./c.out
#include <stdio.h>
#include <bitset>
#include <climits>
#include <iostream>

#define ANAL(x) myprint(#x, x)

void myprint(const char *strv, double v) {
    union {
        double d;
        long long ll;
    } data;

    data.d = v;
    std::bitset<sizeof(double) * CHAR_BIT> bits(data.ll);
    printf("%20s: %.40lf --> ", strv, data.d);
    std::cout << bits << std::endl;
}


int main() {
   ANAL(0.04);
   ANAL(0.1*0.1*4);
   ANAL(0.05+0.04*3);
   ANAL(0.17);
   ANAL(0.05+0.1*0.1*4*3);
}
-----
                0.04: 0.0400000000000000008326672684688674053177 --> 0011111110100100011110101110000101000111101011100001010001111011
           0.1*0.1*4: 0.0400000000000000077715611723760957829654 --> 0011111110100100011110101110000101000111101011100001010001111100
         0.05+0.04*3: 0.1699999999999999844568776552478084340692 --> 0011111111000101110000101000111101011100001010001111010111000010
                0.17: 0.1700000000000000122124532708767219446599 --> 0011111111000101110000101000111101011100001010001111010111000011
    0.05+0.1*0.1*4*3: 0.1700000000000000399680288865056354552507 --> 0011111111000101110000101000111101011100001010001111010111000100
```
其中 `:`和`-->`之间的值是那些变量在内存里真实值，`-->`后面的是那些变量在内存里的01串的值。

可以看到，就是有误差。

# 但是 `atof` 和 `scanf` 可以通过测试
```
$ cat b.cpp && echo '-----' && g++ b.cpp -o b.out && ./b.out
#include <stdio.h>
#include <stdlib.h>

double my_atof(const char *str) {
    return atof(str);
}

int main() {
    bool ok = (0.05 + 3 * my_atof("0.04") < 0.17000000000000001);
    printf(ok ? "test pass\n" : "test fail\n");
    return 0;
}
-----
test pass
```

```
$ cat b.cpp && echo '-----' && g++ b.cpp -o b.out && ./b.out
#include <stdio.h>
#include <stdlib.h>

double my_atof(const char *str) {
    double v;
    sscanf(str, "%lf", &v);
    return v;
}

int main() {
    bool ok = (0.05 + 3 * my_atof("0.04") < 0.17000000000000001);
    printf(ok ? "test pass\n" : "test fail\n");
    return 0;
}
-----
test pass
```

# 搜寻
最终在 https://groups.google.com/forum/#!topic/comp.lang.c/pdYaT-CNwhI 这里看到了答案
```
/*---------------------------------------------------------------------------*/
/* strtod() - convert a string to a double
     */
/*---------------------------------------------------------------------------*/
#include <crt.h>
#include <math.h>
#include <float.h>
#include <ctype.h>
#include <errno.h>
#include <stdlib.h>

double strtod(const char *s, char **endptr)
{
   int digit = 0;
   int expon = 0;
   char sign = 0;
   char esign = 0;
   double fdec = 0;
   double fnum = 0;
   double fpower = 1;
   while((*s == ' ') || (*s == '\t') || *s == '\n' || *s == '\r')
   {
     s++;
   }
   if((*s == '-') || (*s == '+')) sign = *s++;
   while(*s == '0')
   {
     *s++;
   }
   if(*s)
   {
     while(isdigit(*s))
     {
       fnum = (fnum * (double)10) + (double)(*s - '0');
       s++;
       digit = 1;
     }
     if(*s == '.')
     {
       s++;
       while(isdigit(*s))
       {
         fdec = (fdec * (double)10) + (double)(*s - '0');
         fpower *= (double)10;
         s++;
         digit = 1;
       }
       fnum += (fdec / fpower);
     }
     if(digit && (*s == 'e' || *s == 'E'))
     {
       s++;
       if((*s == '-') || (*s == '+')) esign = *s++;
       while(isdigit(*s))
       {
         expon *= 10;
         expon += (*s - '0');
         if(expon >= DBL_MAX_EXP)
         {
           fnum = 1.7976931348623157e+308; //HUGE_VAL;
           expon = 0;
           break;
         }
         *s++;
       }
       fpower = 1;
       while(expon > 100)
       {
         fpower *= (double)1e100;
         expon -= 100;
       }
       while(expon > 10)
       {
         fpower *= (double)1e10;
         expon -= 10;
       }
       while(expon)
       {
         fpower *= (double)10;
         expon --;
       }
       if(esign == '-')
       {
         fnum /= fpower;
       } else {
         fnum *= fpower;
       }
     }
   }
   if(sign == '-') fnum = -fnum;
   if(endptr != NULL) *endptr = (char *)s;
   return(fnum);
}


float strtof(const char *s, char **endptr)
{
     return strtod(s,endptr);
}

double atof(char *p)
{
     return strtod(p,&p);
}
```
可以看到，它的做法是把小数部分变成两个整数相除，于是精度保留得会好一点。我们试试：
```
$ cat b.cpp && echo '-----' && g++ b.cpp -o b.out && ./b.out
#include <stdio.h>
#include <stdlib.h>

double my_atof(const char *str) {
        return (double)4 / (double)100;
}

int main() {
    bool ok = (0.05 + 3 * my_atof("0.04") < 0.17000000000000001);
    printf(ok ? "test pass\n" : "test fail\n");
    return 0;
}
-----
test pass
```

# 分析
我看了下 `0.04` 周围的几个 `double` 值：
```
$ cat c.cpp && echo '-----' && g++ c.cpp -o c.out && ./c.out
#include <stdio.h>
#include <bitset>
#include <climits>
#include <iostream>

union DDD {
    double d;
    long long ll;
} ;

void myprint(double v) {
    DDD data;

    data.d = v;
    std::bitset<sizeof(double) * CHAR_BIT> bits(data.ll);
    printf("%.40lf --> ", data.d);
    std::cout << bits << std::endl;
}

int main() {
    DDD data;

    for (int i = -1; i < 5; i++) {
        data.d = 0.04;
        data.ll += i;
        myprint(data.d);
    }

    return 0;
}
-----
0.0399999999999999938937733645616390276700 --> 0011111110100100011110101110000101000111101011100001010001111010
0.0400000000000000008326672684688674053177 --> 0011111110100100011110101110000101000111101011100001010001111011
0.0400000000000000077715611723760957829654 --> 0011111110100100011110101110000101000111101011100001010001111100
0.0400000000000000147104550762833241606131 --> 0011111110100100011110101110000101000111101011100001010001111101
0.0400000000000000216493489801905525382608 --> 0011111110100100011110101110000101000111101011100001010001111110
0.0400000000000000285882428840977809159085 --> 0011111110100100011110101110000101000111101011100001010001111111
```
结合我们上面对 `0.1 * 0.1 * 4` 的结果，知道结果偏了一个点位:
```
     0.04: 0.0400000000000000008326672684688674053177 --> 0011111110100100011110101110000101000111101011100001010001111011
0.1*0.1*4: 0.0400000000000000077715611723760957829654 --> 0011111110100100011110101110000101000111101011100001010001111100
```