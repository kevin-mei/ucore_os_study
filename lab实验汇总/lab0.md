出于很好奇，我还是准备验证下一下：

网上搜了下，这是gcc编译器的优化； 网上说这个偏移量是在编译时确定的，并不是在运行时。

Gcc和vs2015验证了下，都可以正常编译，运行， gcc的结果是8. vs2015的结果是4， 跟规定的字节对齐方式有关；

```
#include <stdio.h>

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

struct link_entry {
    struct link_entry *prev,*next;
};

struct Page{
    int data;
    struct link_entry node_link;
};

int main()
{
    size_t a = offsetof(struct Page, node_link);
    printf("%ld\n", a);
    return 0;
}

```

本身这个offsetof宏也是存在的，C++定义于头文件 <cstddef>， c 定义于头文件 <stddef.h> 看头文件基本上就能看出来这个宏定义的基本实现版本，有三个，`#define offsetof(s,m) ((size_t)&(((s*)0)->m))`和我们的定义一致
```
//  <stddef.h>  vs2015的头文件
// Define offsetof macro
#if defined(_MSC_VER) && !defined(_CRT_USE_BUILTIN_OFFSETOF)
    #ifdef __cplusplus
        #define offsetof(s,m) ((size_t)&reinterpret_cast<char const volatile&>((((s*)0)->m)))
    #else
        #define offsetof(s,m) ((size_t)&(((s*)0)->m))
    #endif
#else
    #define offsetof(s,m) __builtin_offsetof(s,m)
#endif
```

图形界面的 diff
meld [文件夹]  [文件夹]  

```
syscall(int num, ...) {
...
asm volatile(

)
}
```
这是一个函数调用
这个函数调用所有的系统调用
它都是通过一个宏展开形成相应的函数



.s 汇编语言源程序;  操作: 汇编

.S汇编语言源程序;  操作: 预处理 + 汇编

