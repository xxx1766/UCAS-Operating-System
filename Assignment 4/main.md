# 操作系统 作业 4

## 1.（程序题）
写一个2线程的程序，首先生成一个从 1 到 1000 万的整数数组，然后用两个线程分别计算数组奇数部分和偶数部分的和，并打印出总的和。分别在单核和双核系统上运行该程序，计算加速比。（采用 pthread API）

提示：单核和双核可在虚拟机上配置；pthread 调用方法网上有大量资料，比如 https://www.ibm.com/developerworks/cn/linux/thread/posix_thread1/index.html

### 设计思路与程序代码
程序总共进行 1000 次多线程尝试，计算总时间。
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define MAX_N 10000000
#define MAX_T 100

long arr[MAX_N];

typedef struct {
    long start;
    long end;
    long * arr;
} Param;

void * calc_sum(void * input) {
    Param * param = input;
    long * res = malloc(sizeof(long));

    *res = 0;
    for (long i = param->start; i <= param->end; i += 2) {
        (*res) += param->arr[i];
    }

    return res;
}


long main() {
    int t = MAX_T;
    while (t--) {
            
        pthread_t my_thread;
        for (long i = 1; i <= MAX_N; i++) {
            arr[i] = i;
        }
        Param odd_param = {1, MAX_N, arr};
        Param even_param = {2, MAX_N, arr};
        void * res_addr;
        long odd_res, even_res, res;
        
        pthread_create(&my_thread, NULL, calc_sum, &odd_param);
        res_addr = calc_sum(&even_param);
        even_res = *(long *)res_addr;
        free(res_addr);
        pthread_join(my_thread, &res_addr);
        odd_res = *(long *)res_addr;
        free(res_addr);
        res = odd_res + even_res;
        // printf("result is %ld\n", odd_res + even_res);
    }

    return 0;
}
```

### 运行结果

单核系统：
```shell
$ time ./a.out

real    0m55.342s
user    0m36.065s
sys     0m19.047s
```

双核系统：
```shell
$ time ./a.out

real    0m58.481s
user    1m16.717s
sys     0m15.199s
```

三核系统：
```shell
$ time ./a.out

real    0m56.913s
user    1m18.289s
sys     0m7.455s
```

分析运行结果，无论是单核、双核还是三核系统，其实际使用时间基本相等，加速比可视为 1。

我不太清楚为什么会造成这样的情况……