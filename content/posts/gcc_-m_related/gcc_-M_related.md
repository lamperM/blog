---
title: "GCC '-M' and Related Parameters"
date: 2022-04-26T19:08:22+08:00
---
As we all know, there are huge number of parameters for GCC. With them, we can make many things possible. Now we talk about -M and related ones.
After reading this article, you will know the meaning of there magic parameters. And I will put some little demos follows. Finally, we will see what can they do in really project. Let's go ahead.  
## -M
Output the dependencies of the input source file. Incluing the names of itself and all included files.    
  
**Little Demo**  
Suppose we have source file `tmp.c`:
```c
/* File: tmp.c */
#include <stdio.h> // system header file
#include "tmp.h"   // user defined header file
int main() {
    return 0;
}
```
If we run `gcc -M tmp.c`, we will get messy output like following. Notice that the first two words is object filename and a colon.
```shell
tmp.o: /home/wanglu/demo/tmp.c /usr/include/stdc-predef.h \
 /usr/include/stdio.h \
 /usr/include/x86_64-linux-gnu/bits/libc-header-start.h \
 /usr/include/features.h /usr/include/x86_64-linux-gnu/sys/cdefs.h \
 /usr/include/x86_64-linux-gnu/bits/wordsize.h \
 /usr/include/x86_64-linux-gnu/bits/long-double.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs-64.h \
 /usr/lib/gcc/x86_64-linux-gnu/9/include/stddef.h \
 /usr/lib/gcc/x86_64-linux-gnu/9/include/stdarg.h \
 /usr/include/x86_64-linux-gnu/bits/types.h \
 /usr/include/x86_64-linux-gnu/bits/timesize.h \
 /usr/include/x86_64-linux-gnu/bits/typesizes.h \
 /usr/include/x86_64-linux-gnu/bits/time64.h \
 /usr/include/x86_64-linux-gnu/bits/types/__fpos_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__mbstate_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__fpos64_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__FILE.h \
 /usr/include/x86_64-linux-gnu/bits/types/FILE.h \
 /usr/include/x86_64-linux-gnu/bits/types/struct_FILE.h \
 /usr/include/x86_64-linux-gnu/bits/stdio_lim.h \
 /usr/include/x86_64-linux-gnu/bits/sys_errlist.h /home/wanglu/demo/tmp.h
```
## -MM
Like `-M` but do NOT output system header files.  
  
**Little Demo**  
Use same source file `tmp.c` above.  
Run `gcc -MM tmp.c`. Output dependencies except system header files.
```shell
tmp.o: /home/wanglu/demo/tmp.c /home/wanglu/demo/tmp.h
```
## -MF
Use with `-M` or `-MM`. Specify output dependencies to file instead of STDOUT.  
  
**Little Demo**  
Use same source file `tmp.c` above.  
Run `gcc -MM -MF dp.txt tmp.c`. File `dp.txt` will be created and filled with dependecies of `tmp.c` except system header files.

## -MD
`-MD` is same as `-M -MF <file>`. But the filename is basd on the object file but replacing `.o` with `.d`.   
  
**Little Demo**  
Use same source file `tmp.c` above.  
Run `gcc -MD tmp.c`. File `tmp.d` will be created and filled with all dependecies.
```shell
-rw-rw-r--  1 wanglu wanglu    67 4月  26 19:49 tmp.c
-rw-rw-r--  1 wanglu wanglu  1175 4月  26 20:27 tmp.d
-rw-rw-r--  1 wanglu wanglu    38 4月  26 19:47 tmp.h
```

## -MMD
`-MMD` is same as `-MM -MF <file>`. Also named on object file but replacing `.o` with `.d`.  
  
**Little Demo**  
Use same source file `tmp.c` above.  
Run `gcc -MMD tmp.c`. File `tmp.d` will be created and filled with dependecies except system header files.

## Application
Here is an important question you may ask me: _Why do we struggle to get the dependencies formats? What can they do?_   
If you are familiar with `make` and `Makefile`, aha, that's it!
With the help of M-related parameters, you can easily handle the problem of **tracing header files**.  
Give you a little demo about my point.
```makefile
-include *.d 
%.o:%.c
        $(CC) $(CFLAGS)  $(INCLUDES) $< -c  -MMD -o $@ 
```
Actually, we do two things in order:
1. When complieing source files, we generate dependency files `xxx.d` at the same time.
2. After geting `xxx.d`, we include them in makefile. As its format is exactly the dependency format required by makefile.

## Summary
Hope this article can give you a clear understanding of M-related parameters in GCC. We can sometimes find them in large projects' makefile. It's very useful to automatic build dependency for header files. So try to use them in your current or next project.
  
  
## Reference
1. [GNU GCC options](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)
2. [GCC -M, -MM, -MMD, -MF, -MT](https://programmer.group/gcc-m-mm-mmd-mf-mt.html) 
