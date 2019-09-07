第三部分是关于GCC内联汇编
内联汇编(Inline assembly)
是指在C语言的开发环境中来使用汇编代码，使得C和混编来混在一起来使用，称之为内联汇编；

为什么会有这个需求?
答：是由于我们的C语言不足以完成所有的CPU的指令，特别是有一些特权指令，c语言没有提供操作的函数，比如加载全局描述符表 LGDT，这么一条机器指令 你没法用C语言表述，你只能用汇编来表述，在这种情况下 内联汇编会发生很大的作用，它可以很简单的在C语言里面写出一句话，来把这个功能给完成，这使得我们写的代码更加清楚，更加便于理解；

内联汇编怎么用？

先看一个例子
```
Assembly(*.S)
 movl $0xffff,%eax
 
Inline assembly(.c)
 asm("movl $0xffff, %%eax\n")
```
内联汇编怎么写呢
加了一个asm代表内联汇编的关键字

完整的格式:
```
asm( assembler template // 汇编语句
:output operands (optional) // 输出操作数 
:input operands (optional) // 输入操作数
:clobbers (optional) // 其它被污染的寄存器
);

```

第二个例子
```
Inline assembly(*c)
uint32_t u_cr0;
asm volatile("mov %%cr0, %0\n":"=r"(u_cr0)); // 注意 限制符是 输出操作数
u_cr0|=0x80000000;
asm volatile("movl %0, %%cr0\n"::"r"(u_cr0)); 、// 注意 限制符是 输入操作数

Generated assembly code(*.s):
movl %cr0,%ebx
mov %ebx,12(%esp)
orl $-2147483648, 12(%esp)
movl 12(%esp),%eax
movl %eax, %cr0
```

并没有看到有Cr0这个寄存器，暂时不懂这个寄存器是啥？

这段代码要达成的操作：CR0寄存器的某一个bit给置位

**volatile 是不需要编译器做进一步的优化，don't touch my code** 
调整顺序那这个%0 代表的是第一个用到的寄存器,这个r代表是任意寄存器的意思

首先把cr0寄存器的内容，读到这个%0寄存器里面去
然后给u_cr0赋上任意寄存器的值，这里r特指%0？也就是附上了cr0寄存器的值; 
接着对u_cr0进行置位操作
把某一个位置成1，或了一下 相当于将第31位(从0开始)置成1
最后一句再把u_cr0写回到%0寄存器里，然后%0寄存器再复制给cr0寄存器
完成对cr0寄存器的置位操作

第二段的汇编代码
CR0寄存器赋给ebx
ebx给到一个12(%esp) 这实际上是代表一个局部变量
然后到orl，这是一个或操作，来完成了对某一个bit的置1
再把这个12(%esp)局部变量里面的值赋给EAX寄存器
最后把EAX赋给CR0；






更复杂的例子
```
long __res,arg1=2,arg2=22,arg3=222,arg4=233;
__asm__ volatile("int $0x80"
:"=a"(__res) // 输出操作数
:"0"(11),"b"(arg1),"c"(arg2),"d"(arg3),"S"(arg4)
); // 输入操作数

...						Constraints
movl$11, %eax			a=%eax
movl-28(%ebp),%ebx		b=%ebx
movl-24(%ebp),%ecx		c=%ecx
movl-20(%ebp),%edx		d=%edx
movl-16(%ebp),%esi		S=%esi
int $0x80				D=%edi
movl%edi,-12(ebp)		U=same as the first
```

这些局部变量赋给相应的寄存器
然后调用int 80这么一个指令
产生一个软中断
最后这个软中断返回的值，会再赋给EBP，最终赋给`__res`


Gcc Manual 6.41-6.43
Inline assembly for x86 in linux:

https://www.ibm.com/developerworks/library/l-ia/index.html  (无法访问)