# 操作系统 作业 13

## 1
请在 Unix 环境下实现如下功能，提交程序和执行结果：
1. 申请一个 `uint32_t` 类型 65536（64K）项的数组（4 KB × 64 = 256 KB，我们相信各位同学的笔记本使用的页大小为 4 KB，而没有使用 linux 的大页机制）；
2. 使用 `mprotect` 函数对申请的页设置只读；
3. 将数组中第 0、1K、2K...63K 项依次赋值为 1、2、3、...、64；
4. 注册 `SIGSEGV` 信号处理函数（建议使用 `sigaction`），在数据访问发生错误时，让程序继续执行，并且打印这 64 次访存的 trace。

**提示**：
1. https://linux.die.net/man/2/sigaction
2. https://linux.die.net/man/3/sigemptyset
3. `siginfo_t` 包含了地址信息

### 解
代码如下：
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>
#include <stdint.h>
#include <stdlib.h>
#include <malloc.h>
#include <sys/mman.h>

int pagesize, array_size;

void sigsegv_sigaction(int signal_number, siginfo_t * siginfo, void * ucontext) {
    printf("SIGSEGV: try to write %16p which is read-only\n", siginfo->si_addr);
    mprotect(siginfo->si_addr, sizeof(int), PROT_READ | PROT_WRITE);
}

struct sigaction my_sigaction = {
    .sa_sigaction = sigsegv_sigaction,
    .sa_flags = SA_SIGINFO,
};

int main() {
    // requirement 1
    pagesize = sysconf(_SC_PAGE_SIZE);
    printf("Pagesize is %d\n", pagesize);
    array_size = 65536 * sizeof(uint32_t);
    uint32_t * array = memalign(pagesize, array_size);

    // requirement 2
    mprotect(array, array_size, PROT_READ);

    // requirement 4
    sigaction(SIGSEGV, &my_sigaction, NULL);

    // requirement 3
    for (int i = 0; i < 64; i++) {
        printf("try: array[%dK] = %d\n", i, i+1);
        array[i * 1024] = i + 1;
    }
    return 0;
}
```


运行结果如图，
```shell
$ gcc virtual_memory.c 
ceba@CEBA-X1C:~/code$ ./a.out 
Pagesize is 4096
try: array[0K] = 1
SIGSEGV: try to write   0x7f39824d0000 which is read-only
try: array[1K] = 2
SIGSEGV: try to write   0x7f39824d1000 which is read-only
try: array[2K] = 3
SIGSEGV: try to write   0x7f39824d2000 which is read-only
try: array[3K] = 4
SIGSEGV: try to write   0x7f39824d3000 which is read-only
try: array[4K] = 5
SIGSEGV: try to write   0x7f39824d4000 which is read-only
try: array[5K] = 6
SIGSEGV: try to write   0x7f39824d5000 which is read-only
try: array[6K] = 7
SIGSEGV: try to write   0x7f39824d6000 which is read-only
try: array[7K] = 8
SIGSEGV: try to write   0x7f39824d7000 which is read-only
try: array[8K] = 9
SIGSEGV: try to write   0x7f39824d8000 which is read-only
try: array[9K] = 10
SIGSEGV: try to write   0x7f39824d9000 which is read-only
try: array[10K] = 11
SIGSEGV: try to write   0x7f39824da000 which is read-only
try: array[11K] = 12
SIGSEGV: try to write   0x7f39824db000 which is read-only
try: array[12K] = 13
SIGSEGV: try to write   0x7f39824dc000 which is read-only
try: array[13K] = 14
SIGSEGV: try to write   0x7f39824dd000 which is read-only
try: array[14K] = 15
SIGSEGV: try to write   0x7f39824de000 which is read-only
try: array[15K] = 16
SIGSEGV: try to write   0x7f39824df000 which is read-only
try: array[16K] = 17
SIGSEGV: try to write   0x7f39824e0000 which is read-only
try: array[17K] = 18
SIGSEGV: try to write   0x7f39824e1000 which is read-only
try: array[18K] = 19
SIGSEGV: try to write   0x7f39824e2000 which is read-only
try: array[19K] = 20
SIGSEGV: try to write   0x7f39824e3000 which is read-only
try: array[20K] = 21
SIGSEGV: try to write   0x7f39824e4000 which is read-only
try: array[21K] = 22
SIGSEGV: try to write   0x7f39824e5000 which is read-only
try: array[22K] = 23
SIGSEGV: try to write   0x7f39824e6000 which is read-only
try: array[23K] = 24
SIGSEGV: try to write   0x7f39824e7000 which is read-only
try: array[24K] = 25
SIGSEGV: try to write   0x7f39824e8000 which is read-only
try: array[25K] = 26
SIGSEGV: try to write   0x7f39824e9000 which is read-only
try: array[26K] = 27
SIGSEGV: try to write   0x7f39824ea000 which is read-only
try: array[27K] = 28
SIGSEGV: try to write   0x7f39824eb000 which is read-only
try: array[28K] = 29
SIGSEGV: try to write   0x7f39824ec000 which is read-only
try: array[29K] = 30
SIGSEGV: try to write   0x7f39824ed000 which is read-only
try: array[30K] = 31
SIGSEGV: try to write   0x7f39824ee000 which is read-only
try: array[31K] = 32
SIGSEGV: try to write   0x7f39824ef000 which is read-only
try: array[32K] = 33
SIGSEGV: try to write   0x7f39824f0000 which is read-only
try: array[33K] = 34
SIGSEGV: try to write   0x7f39824f1000 which is read-only
try: array[34K] = 35
SIGSEGV: try to write   0x7f39824f2000 which is read-only
try: array[35K] = 36
SIGSEGV: try to write   0x7f39824f3000 which is read-only
try: array[36K] = 37
SIGSEGV: try to write   0x7f39824f4000 which is read-only
try: array[37K] = 38
SIGSEGV: try to write   0x7f39824f5000 which is read-only
try: array[38K] = 39
SIGSEGV: try to write   0x7f39824f6000 which is read-only
try: array[39K] = 40
SIGSEGV: try to write   0x7f39824f7000 which is read-only
try: array[40K] = 41
SIGSEGV: try to write   0x7f39824f8000 which is read-only
try: array[41K] = 42
SIGSEGV: try to write   0x7f39824f9000 which is read-only
try: array[42K] = 43
SIGSEGV: try to write   0x7f39824fa000 which is read-only
try: array[43K] = 44
SIGSEGV: try to write   0x7f39824fb000 which is read-only
try: array[44K] = 45
SIGSEGV: try to write   0x7f39824fc000 which is read-only
try: array[45K] = 46
SIGSEGV: try to write   0x7f39824fd000 which is read-only
try: array[46K] = 47
SIGSEGV: try to write   0x7f39824fe000 which is read-only
try: array[47K] = 48
SIGSEGV: try to write   0x7f39824ff000 which is read-only
try: array[48K] = 49
SIGSEGV: try to write   0x7f3982500000 which is read-only
try: array[49K] = 50
SIGSEGV: try to write   0x7f3982501000 which is read-only
try: array[50K] = 51
SIGSEGV: try to write   0x7f3982502000 which is read-only
try: array[51K] = 52
SIGSEGV: try to write   0x7f3982503000 which is read-only
try: array[52K] = 53
SIGSEGV: try to write   0x7f3982504000 which is read-only
try: array[53K] = 54
SIGSEGV: try to write   0x7f3982505000 which is read-only
try: array[54K] = 55
SIGSEGV: try to write   0x7f3982506000 which is read-only
try: array[55K] = 56
SIGSEGV: try to write   0x7f3982507000 which is read-only
try: array[56K] = 57
SIGSEGV: try to write   0x7f3982508000 which is read-only
try: array[57K] = 58
SIGSEGV: try to write   0x7f3982509000 which is read-only
try: array[58K] = 59
SIGSEGV: try to write   0x7f398250a000 which is read-only
try: array[59K] = 60
SIGSEGV: try to write   0x7f398250b000 which is read-only
try: array[60K] = 61
SIGSEGV: try to write   0x7f398250c000 which is read-only
try: array[61K] = 62
SIGSEGV: try to write   0x7f398250d000 which is read-only
try: array[62K] = 63
SIGSEGV: try to write   0x7f398250e000 which is read-only
try: array[63K] = 64
SIGSEGV: try to write   0x7f398250f000 which is read-only
```

参考资料：
* https://linux.die.net/man/3/sigaction
* https://linux.die.net/man/3/getcontext
* https://man7.org/linux/man-pages/man2/mprotect.2.html
* https://man7.org/linux/man-pages/man3/posix_memalign.3.html


<style>
a {
    color: black;
}
</style>