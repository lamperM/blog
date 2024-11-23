---
title: "C 语言的特点与难点"
date: 2023-03-09T17:18:57+08:00
tags: [c]
categories: ["C Language"]
---

## 二维字符数组 char *strs[]
定义一个二位字符数组的语句是：
```c
char *KW[] = {"return", "if", "else"};
char **KW = {"return", "if", "else"}; // 错误
char KW[][] = {"return", "if", "else"}; // 错误

```

这种情况可以怎么理解呢？`{"return", "if", "else"}`是一些字符串常量，存储在Elf文件的常量区。

那么要KW变量是一个二维数组，每个二位数组要存好这些个字符串分别的地址。

- `char *KW[]`就是这么做的，[]编译器可以推断长度是3。
- `char **kw`错误的原因是这不是个二维数组啊，没有分配长度为3的数据空间。
- `char KW[][]`错误的原因是这种定义方式并不适用于字符串常量的情况，是想要让字符串数组存储到变量的栈中，而不仅仅是地址。然而在直接存储字符串的情况下，又没有指定每个字符串的长度，这样编译器没办法分配空间。只能改成这样 `char KW[][6]`

## 指针的数组 or 指向数组的指针?

``` 
>> int (*p)[10]   p是指针, 指向长度为10的数组. 加括号是为了强调p是一个指针, 区别包含10个指针的array.
>> int *(p[10])   p是数组, 它的元素类型是int *, 加括号是为了强调p是数组.
>> int *p[10]     等效于int *(p[10])
```

## 程序的内存分布

我们写的高级语言代码最终会被编译成可执行文件被操作系统加载、执行。
ELF文件是由一个个section组成的，那么程序里的变量、指令都是如何排布的呢？

| 地址空间各部分 | 内容                        | 是否与可执行文件相对应      |
|---------|---------------------------|------------------|
| 代码段     | 所有的可执行代码, 属性一般为只读         | 是, 加载时直接映射       |
| 数据段     | 初始化非0的全局变量和局部static变量     | 是, 加载时直接映射       |
| bss段    | 未初始化或初始化0的全局变量和局部static变量 | 是, 加载时需要清空       |
| 栈       | 局部变量, 参数传递等               | 无, OS分配空间, 编译器维护 |
| 堆       | 动态申请的空间                   | 无, OS分配空间        |
| 常量区     | 定义的常量字符串等                 | 是, 有时和代码段合并到一起   |

> 有的人喜欢说“静态区”这个概念，我也一直被忽悠不理解什么叫静态区。实际上就是表示
> 存储函数内部static变量的区域，**本质上属于数据段的一部分**。

> static声明的全局变量，对于编译器来说有什么区别对待?
>
> 除了预编译时不会建立符号外没有区别。

> 全局变量和局部变量可以重名吗？
>
> 可以重名。因为局部变量用栈来相对索引就可以，不需要符号，也就不会产生冲突。
> 而全局变量都是用符号来索引的。**默认访问的是局部变量，如果希望访问全局变量，
> 需要使用`::val += 1;`的语法。**

## 灵活数组成员 Flexiable Array Member

1. C99支持的特性，目的是为了节约空间（灵活数组成员本身不占空间，见下输出）
2. 使用灵活成员**可以用于实现string结构，长度动态分配**  
1. 灵活成员必须是结构的**最后一个**成员，且此结构体不能只包含一个灵活成员（最少俩）
3. 使用了灵活成员后，就不能用结构体之间直接"="赋值了，改用`memcpy()`
4. 含有灵活成员的结构体不能嵌入其他结构体中

```c
#include <stdio.h>
#include <stdlib.h>

struct st {
  int memb1;
  int memb2;
  int memb3[]; // flexiable array member
};

int main(void)
{

  printf("sizeof struct st: %ld\n", sizeof(struct st));

  /* instantiate */
  struct st *st1; // 只能动态分配
  int memb3_len = 10;

  st1 = malloc(sizeof(struct st) + memb3_len*sizeof(int));

  /* use */
  for (int i = 0; i < memb3_len; i++)
    st1->memb3[i] = i;

  for (int i = 0; i < memb3_len; i++)
    printf("st1->mem3[%d] = %d\n", i, st1->memb3[i]);


  free(st1);
  return 0;
}
```

输出结果:
```
sizeof struct st: 8
st1->mem3[0] = 0
st1->mem3[1] = 1
st1->mem3[2] = 2
st1->mem3[3] = 3
st1->mem3[4] = 4
st1->mem3[5] = 5
st1->mem3[6] = 6
st1->mem3[7] = 7
st1->mem3[8] = 8
st1->mem3[9] = 9
```



## 基础架构

```
// 函数指针
>> int (*f)(int)  说明f是一个指向函数的指针, 加括号为了区别返回值为int*的函数
>> f = function;  函数指针的赋值
>> (*f)(x)        函数指针指向函数的调用, 可简化为f(x). 但是容易将f误认为是函数.
    
// 函数指针的数组
>> int (*(f[10])) (int)  f是数组,元素为10个函数指针. 内层括号说明f是数组,外层括号说明元素类型是函数指针
>> int (*f[10]) (int)    与上面等效. 但外层括号不能省略
>> f[0] = function()     赋值
>> (*f[0])()             指向函数的调用, 可简化为f[0]()
    
// 返回函数指针的函数
>> void (*signal(int sig, ...))(int);  signal是一个函数, 参数有sig.... 它的返回值是一个函数指针, 指向任意返回值为void, 参数为int的函数.
```


## `typedef`帮助理解函数指针

`signal()`是一个系统调用, 用于告诉系统, 当某种特定"软件中断"发生时调用*特定的程序*. 它的真正名称应当是: *Call that routine when the interrupt comes in*.

看`signal()`的原型, 非常复杂. 根据上面基础架构的铺垫, 可以看出`signal()`的返回值是函数指针, 同时它的参数也是一个函数指针. 且这两个函数指针所指向函数的返回值和参数相同.

```c
void (*signal(int sig, void(*func)(int)))(int);
```

可以借用`typedef`表示通用部分.

```c
typedef void (*sighandler_t)(int);
```

而后`signal`的声明就是人能看懂的了:

```c
sighandler_t signal(int signum, sighandler_t handler);
```


# C语言标准

我们在使用C语言编程时很少有人告诉我们C语言各个标准的情况，于是我们在看见一些函数标定支持的C标准（例如仅支持C99及以后），内心不会有什么波澜。

我们常见这些C标准：K&R C、ANSI C、ISO C、C89、C99、C11、C18。让我们补充点可能很少使用的知识吧。

## 什么是K&R C？

1978年，丹尼斯•里奇（Dennis Ritchie）和布莱恩•柯林汉（Brian ernighan）合作出版了《C程序设计语言》的第一版。书中介绍的C语言标准也被称作“K&R C”。

最初的C标准与我们现在用的有较大差别，例如它竟然还不支持void类型！

## 什么是ANSI C、ISO C、C89、C90标准？

随着C语言使用得越来越广泛，出现了许多新问题，人们日益强烈地要求对C语言进行标准化。1983年，美国国家标准协会（ANSI）组成了一个委员会，X3J11，为了创立 C 的一套标准。经过漫长而艰苦的过程，该标准于1989年完成，这个版本的语言经常被称作ANSI C，或有时称为C89（为了区别C99）。在1990年，ANSI C标准（带有一些小改动）被美国国家标准协会（ANSI）采纳为ISO/IEC 9899:1990。这个版本有时候称为C90或者ISO C。综上，ANSI C、ISO C、C89、C90其实是同一种标准。

这一版本的C就更接近我们平常使用的C了，大部分特性都引入了。

## 什么是C99标准？

2000年3月，ANSI 采纳了 ISO/IEC 9899:1999 标准。这个标准通常指C99。

C99我们最常使用的新特性是：在源代码的中间位置声明变量。

## 什么是C11标准？

C11标准是C语言标准的第三版（2011年由ISO/IEC发布），前一个标准版本是C99标准。与C99相比，C11有哪些变化呢？

```c
1、 对齐处理：alignof(T)返回T的对齐方式，aligned_alloc()以指定字节和对齐方式分配内存，头文件<stdalign.h>定义了这些内容。
2、 _Noreturn：_Noreturn 是个函数修饰符，位置在函数返回类型的前面，声明函数无返回值，有点类似于gcc的__attribute__((noreturn))，后者在声明语句尾部。
3、 _Generic：_Generic支持轻量级范型编程，可以把一组具有不同类型而却有相同功能的函数抽象为一个接口。
4、 _Static_assert()：_Static_assert()，静态断言，在编译时刻进行，断言表达式必须是在编译时期可以计算的表达式，而普通的assert()在运行时刻断言。
5、安全版本的几个函数：gets_s()取代了gets()，原因是后者这个I/O函数的实际缓冲区大小不确定，以至于发生常见的缓冲区溢出攻击，类似的函数还有其它的。
6、 fopen()新模式：fopen()增加了新的创建、打开模式“x”，在文件锁中比较常用。
7、 匿名结构体、联合体。
8、 多线程：头文件<threads.h>定义了创建和管理线程的函数，新的存储类修饰符_Thread_local限定了变量不能在多线程之间共享。
9、 _Atomic类型修饰符和头文件<stdatomic.h>。
10、改进的Unicode支持和头文件<uchar.h>。
11、quick_exit()：又一种终止程序的方式，当exit()失败时用以终止程序。
12、复数宏，浮点数宏。
13、time.h新增timespec结构体，时间单位为纳秒，原来的timeval结构体时间单位为毫秒。
```

## 什么是C18标准？

C18也称C17是于2018年6月发布的 ISO/IEC 9899:2018 的非正式名称，也是目前（截止到2020年6月）为止最新的 C语言编程标准，被用来替代 C11 标准。

C17 没有引入新的语言特性，只对 C11 进行了补充和修正。

​                                                                                      

## 如何查看自己程序的C标准版本？

使用宏\__STDC_VERSION__可以输出当前使用的C标准版本，是一个长整型：

```c
printf("C std version:%ld\n", __STDC_VERSION__);
```

值与标准的对应关系：

| 标准 | 宏                         |
| ---- | -------------------------- |
| C94  | \__STDC_VERSION__= 199409L |
| C99  | \__STDC_VERSION__= 199901L |
| C11  | \__STDC_VERSION__= 201112L |
| C18  | \__STDC_VERSION__= 201710L |

​                                                                                                                           

## 如何指定按照某个标准执行编译？

以下的介绍只针对GCC，我没有用过别的编译器。

GCC中可以添加`--std=xxx`来指定C标准版本，常用的情况如下：

```c
-std=c11             Conform to the ISO 2011 C standard
-std=c89             Conform to the ISO 1990 C standard
-std=c90             Conform to the ISO 1990 C standard
-std=c99             Conform to the ISO 1999 C stand
    
    
-std=gnu11           Conform to the ISO 2011 C standard with GNU extensions
-std=gnu89           Conform to the ISO 1990 C standard with GNU extensions
-std=gnu90           Conform to the ISO 1990 C standard with GNU extensions
-std=gnu99           Conform to the ISO 1999 C standard with GNU extensions
```                                                                                            

> 默认情况下，我电脑上的`gcc 5.4.0`使用`-std-gnu11`


## 参考目录

https://blog.csdn.net/zhengnianli/article/details/87387268

[C Dialect Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html#C-Dialect-Options)



# 含糊不清的符号扩展

## 问题出在哪？

下面一段代码会输出什么呢？

```c
char c = 0xff;

if (c == 0xff)
    printf("successful\n");
else
    printf("failed\n");
```

答案是取决于不同的编译器设定：

- 当编译器将char识别为signed char时，该判断会失败。因为常数0xff被识别为int类型，所以编译器首先要对c进行符号扩展，判断语句`c == 0xff`此时等价于`(int)c == 0xff`。而对于signed char类型是扩展其最高位，即`(int)c=0xffffffff`，if判断失败。
- 当编译器将char识别为unsigned char时，判断成功。对于unsigned char类型总是扩展0。

> 注：gcc可通过添加编译参数 -fsigned-char/ -funsigned-char来指定编译器如何识别char

同样的问题也存在与位域(bitfiled)中，详见-fsigned-bitfields/-funsigned-bitfields参数。                                               


## 如何避免？

在使用char类型时，根据情况写清楚unsigned/signed char就ok

```c
unsigned char c = 0xff

if (c == 0xff)
    printf("successful\n");
else
    printf("failed\n");
```


# 右移和除法

你是否有听说过有符号数不能使用右移操作(`>>`)来代替除法？ 这篇短文会向你证明它，并尝试向你解释为什么。

## Logical Shift .vs. Arithmetic Shift

若你现在有二进制数`x=1110B`，对其施加右移操作，请问高位填0还是填1？

**逻辑移位**不管造成的影响，总是用0来填充移位操作产生的空缺。但是这样简单的想法在一些情况总会出错。例如若上述x是有符号数，那么简单的填0就会造成错误，起码正负号出错了。

**算数移位**支持有符号数的移位操作，在移位后使用<u>符号位</u>进行填充，结合补码的表示方法，就能实现正确的负数移位操作。

总结来说：在有符号的场景下，使用算数位移；如果你能保证移位操作是无符号的，那么用逻辑位移也无妨.

x86汇编代码中，`shr`代表逻辑右移指令，`sar`代表算数右移指令，我们可以通过以下C代码及其反汇编的结果来更好的理解逻辑移位和算数移位：

```c
#include <stdlib.h>
#include <stdio.h>
    
signed int x = -3;
unsigned int y = 3;

int main()
{
    x >>= 1;
    y >>= 1;
    return 0;
}

```

```assembly
x:
        .long   -3
y:
        .long   3
main:
        push    rbp
        mov     rbp, rsp
        mov     eax, DWORD PTR x[rip]
        sar     eax
        mov     DWORD PTR x[rip], eax
        mov     eax, DWORD PTR y[rip]
        shr     eax
        mov     DWORD PTR y[rip], eax
        mov     eax, 0
        pop     rbp
        ret
```

https://godbolt.org/z/K4M4Ko4c7

## 实践出真知

在我作为一个初级程序员的认知中，`/2`和`>>1`是等价的，甚至一起还听说过后者能够优化代码的效率。但是今天我要告诉你， Definitely wrong!

或许在遥远的古代，我们使用位移操作真的能够对代码进行加速，但是当下编译器已经足够聪明，如果你真的动手反汇编"`/2`"的代码，那么你就会知道编译器已经替你优化为了位移操作。

更糟糕的是，我们要避免使用移位操作来实现除法或者乘法，不仅仅是因为这两者等价，实际上，他们并不是等价的！并且会造成错误！

考虑如下的C语言代码：

```c
#include <stdlib.h>
#include <stdio.h>

signed int x = -3;
signed int y = -3;

int main()
{
    x >>= 1;
    y /= 2;
    return 0;
}

```

他们的汇编代码是相同的吗？这里还是拿X86汇编举例：

```assembly
; Following is ‘x >>= 1’
mov     eax, #-3  ;x
sar     eax
mov     x, eax
; Following is ‘y/= 2’
mov     eax, #-3  ;y
mov     edx, eax
shr     edx, 31
add     eax, edx
sar     eax
mov     y, eax
```

注意：以上的汇编代码省去了一些我认为无关紧要的操作，并不是完全正确的，但是足够表达他们的差别了。

可以看出，除法比移位多了一步`shr edx, 31`过程，下面会探讨这个。

还有一件使你震惊的事件，`x`, `y`的值最终是**不同**的！是的，正是因为那条看似“多余”的`shr`指令。

## 为什么结果不同

首先，我们可以确定的一件事是：编译器真的帮我们将除法操作优化为移位。所以，再也不要说你的代码中使用`>>`来替代除法是为了增加执行效率了。

让我们来解释下为什么两者的结果是不同的。

首先，`sar`指令在x86指令集中表示算数右移，这个是我们熟悉的，那么`-3`进行算数右移后的结果就是`-2`.  意味着`>>`是向负无穷舍入的.

那么除法操作又是在干什么呢? 它是将原值加上其符号位.Demo中使用的数据类型是32位`int`.

```assembly
shr     edx, 31
add     eax, edx
```

这样做必然改变了原值啊，动手算一下就会知道，`-3/2`的结果为`-1`. 并且只有负奇数会受影响，对于正数，其符号为0；对于负偶数，其补码的最低位必为0，刚加上的1会被下一步的算数右移丢弃，不对高位产生影响。  

Aha, 差别就是向负无穷舍弃还是向0舍弃，一时间竟然不知道哪个是正确的了。

## 我们应该如何做

根据最新的[C语言标准草案]([ISO/IEC 9899:201x (open-std.org)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1336.pdf)) 6.5.7章节，负数的右移操作是implementation-defined，即取决于具体的实现：

> The result of E1 >> E2 is E1 right-shifted E2 bit positions. If E1 has an unsigned type or if E1 has a signed type and a nonnegative value, the value of the result is the integral part of the quotient of E1 / 2E2. If E1 has a signed type and a negative value, the resulting value is implementation-defined.

因此，理论上它依赖于实现。所以我们在实际应用中为了程序的可移植性，应当避免对有符号数使用移位操作。除非你能确定它的值一定是非负数，在此情况下，请将它用无符号类型来声明。  

对于除法操作，标准中的6.5.5章节规定了，除法操作总是向0舍入. 非常好！

> When integers are divided, the result of the / operator is the algebraic quotient with any fractional part discarded.

检查你的代码，恢复所有的“优化”乘除法的行为吧！

