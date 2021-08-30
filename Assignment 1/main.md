# 操作系统 作业 1

### 题目
请在一个posix兼容的环境（unix，linux，windows cmd、mac等）编译执行附件小程序，并试着分析每个变量所属的段（section），可以用 objdump 等进行验证。

```c
#include <stdio.h>
#include <stdlib.h>

char *myname="Bao Yungang";
char gdata[128];
char bdata[16] = {1,2,3,4};
main() {
	char * ldata[16];	
	char * ddata;

	ddata = malloc(16);
	printf("gdata: %llX\nbdata:%llX\nldata:%llx\nddata:%llx\n",
		gdata,bdata,ldata,ddata);
	free(ddata);
}
```

### 解答
* `char *` 类型指针 `myname` 位于数据段的 `.data` 节中。
  * 经过初始化的全局变量，位于数据段的 `.data` 节中，同时存在于运行时内存和可执行文件。
* 字符串常量 `"Bao Yungang"` 位于数据段的 `.rodata` 节中。
  * 只读数据，位于数据段的 `.rodata` 节（Read Only Data）中，同时存在于运行时内存和可执行文件。
* 字符数组 `gdata` 位于数据段的 `.bss` 节中
  * 未初始化、初始化为全 0 的数据，位于数据段的 `.bss` 节中，仅运行时存在于内存中，不存在于可执行文件中。
* 字符数组 `bdata` 位于数据段的 `.data` 节中。
  * 经过初始化的全局变量，位于数据段的 `.data` 节中，同时存在于运行时内存和可执行文件。
* `char *` 类型指针数组 `ldata` 位于栈中。
  * 局部变量，位于栈中，运行时分配内存空间。
* `char *` 类型指针 `ddata` 位于栈中。
  * 局部变量，位于栈中，运行时分配内存空间。
* `malloc(16)` 申请的内存空间位于堆中。
  * 通过 `malloc` 分配的空间，位于堆中，运行时通过库函数分配
* 字符串常量 `"gdata: %llX\nbdata:%llX\nldata:%llx\nddata:%llx\n"` 位于数据段的 `.rodata` 节中。
  * 只读数据，位于数据段的 `.rodata` 节中，同时存在于运行时内存和可执行文件。

部分通过 `objdump` 得到的结果：
```
Contents of section .rodata:
 2000 01000200 00000000 42616f20 59756e67  ........Bao Yung
 2010 616e6700 00000000 67646174 613a2025  ang.....gdata: %
 2020 6c6c580a 62646174 613a256c 6c580a6c  llX.bdata:%llX.l
 2030 64617461 3a256c6c 780a6464 6174613a  data:%llx.ddata:
 2040 256c6c78 0a00                        %llx..          

Contents of section .data:
 4000 00000000 00000000 08400000 00000000  .........@......
 4010 01020304 00000000 00000000 00000000  ................
 4020 08200000 00000000                    . ......        
```

其中：
* `2008 ~ 2013` 为字符串常量 `"Bao Yungang"`
* `2018 ~ 2045` 为字符串常量 `"gdata: %llX\nbdata:%llX\nldata:%llx\nddata:%llx\n"`
* `4010 ~ 401F` 为数组 `bdata`
* `4020 ~ 4007` 为 `char *` 类型指针 `myname`

