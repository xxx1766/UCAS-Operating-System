# 操作系统 作业 6

## 同步作业 1

写一个两线程程序,两线程同时向一个数组分别写入 1000 万以内的奇数和偶数，过程中
共用一个偏移量。写完后打印出数组相邻两个数的最大绝对差值。

请分别按下列方法完成一个**不会丢失数据**的程序：
1. 请用 Peterson 算法实现上述功能。
2. 学习了解 `pthread_mutex_lock/unlock()` 函数的功能，并实现上述程序功能。
3. 学习了解 `atomic_add()`（`_sync_fetch_and_add()` for gcc 4.1+）函数，实现上述程序功能。

作业要求：
1. 要求方法 1 和方法 2 每次进入临界区后，执行一百次操作后离开临界区。
2. 请找一个双核系统测试，分别列出三种方式的执行时间。

### 解答
#### 1. 使用 Peterson 算法实现

程序代码 `peterson.c`：
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <stdbool.h>

#define MAX_N 10000000
#define N_THREAD 2

int data_index = 0;
int data[MAX_N];

// Control Variables for Peterson's Algorithm
int peterson_turn;
bool peterson_flag[N_THREAD];

void * func_thread_0(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        if (i % 200 == 0) {         // Lock when 100 operation starts
            peterson_flag[0] = true;
            peterson_turn = 1;
            while (peterson_flag[1] && peterson_turn == 1);
        }
        data[data_index] = i;       // even
        data_index ++;
        if (i % 200 == 198) {       // Unlock when 100 operation ends
            peterson_flag[0] = false;
        }
    }
    return NULL;
}

void * func_thread_1(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        if (i % 200 == 0) {         // Lock when 100 operation starts
            peterson_flag[1] = true;
            peterson_turn = 0;
            while (peterson_flag[0] && peterson_turn == 0);
        }
        data[data_index] = i + 1;   // odd
        data_index ++;
        if (i % 200 == 198) {       // Unlock when 100 operation ends
            peterson_flag[1] = false;
        }
    }
    return NULL;
}

int main() {
    pthread_t thread_0, thread_1;
    pthread_create(&thread_0, NULL, func_thread_0, NULL);
    pthread_create(&thread_1, NULL, func_thread_1, NULL);
    pthread_join(thread_0, NULL);
    pthread_join(thread_1, NULL);

    int max_delta = 0;
    for (int i = 1; i < MAX_N; i++) {
        int delta = abs(data[i] - data[i - 1]);
        if (delta > max_delta) {
            max_delta = delta;
        }
    }

    printf("Max delta = %d\n", max_delta);

    return 0;
}
```

#### 2. 使用 `pthread_mutex_lock/unlock()`

程序代码 `mutex.c`：
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define MAX_N 10000000
#define N_THREAD 2

int data_index = 0;
int data[MAX_N];

// Mutex Lock
pthread_mutex_t data_index_mutex = PTHREAD_MUTEX_INITIALIZER;

void * func_thread_0(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        if (i % 200 == 0) {         // Lock when 100 operation starts
            pthread_mutex_lock(&data_index_mutex);
        }
        data[data_index] = i;       // even
        data_index ++;
        if (i % 200 == 198) {       // Unlock when 100 operation ends
            pthread_mutex_unlock(&data_index_mutex);
        }
    }
    return NULL;
}

void * func_thread_1(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        if (i % 200 == 0) {         // Lock when 100 operation starts
            pthread_mutex_lock(&data_index_mutex);
        }
        data[data_index] = i + 1;   // odd
        data_index ++;
        if (i % 200 == 198) {       // Unlock when 100 operation ends
            pthread_mutex_unlock(&data_index_mutex);
        }
    }
    return NULL;
}

int main() {
    pthread_t thread_0, thread_1;
    pthread_create(&thread_0, NULL, func_thread_0, NULL);
    pthread_create(&thread_1, NULL, func_thread_1, NULL);
    pthread_join(thread_0, NULL);
    pthread_join(thread_1, NULL);

    int max_delta = 0;
    for (int i = 1; i < MAX_N; i++) {
        int delta = abs(data[i] - data[i - 1]);
        if (delta > max_delta) {
            max_delta = delta;
        }
    }

    printf("Max delta = %d\n", max_delta);

    return 0;
}
```

#### 3. 使用原子加操作 `__sync_fetch_and_add()`

`__sync` 系列函数是 GCC 为原子内存访问提供的内建函数，共有12个。这个实验中使用的是 `__sync_fetch_and_add()`，类似于 `i++`。

这一系列内建函数应当被被 `__atomic` 系列内建函数替代。

程序代码 `__sync.c`：
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define MAX_N 10000000
#define N_THREAD 2

int data_index = 0;
int data[MAX_N];

void * func_thread_1(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        int temp = __sync_fetch_and_add(&data_index, 1);
        data[temp] = i;        // even
    }
    return NULL;
}

void * func_thread_2(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        int temp = __sync_fetch_and_add(&data_index, 1);
        data[temp] = i + 1;    // odd
    }
    return NULL;
}

int main() {
    pthread_t thread_1, thread_2;
    pthread_create(&thread_1, NULL, func_thread_1, NULL);
    pthread_create(&thread_2, NULL, func_thread_2, NULL);
    pthread_join(thread_1, NULL);
    pthread_join(thread_2, NULL);

    int max_delta = 0;
    for (int i = 1; i < MAX_N; i++) {
        int delta = abs(data[i] - data[i - 1]);
        if (delta > max_delta) {
            max_delta = delta;
        }
    }

    printf("Max delta = %d\n", max_delta);

    return 0;
}
```

#### 4. 使用原子加操作 `__atomic_fetch_add()`

`__atomic` 系列函数是 GCC 为原子内存访问提供的一套新的内建函数，用于替代 `__sync` 系列内建函数，提供的函数数量也更多。

这个实验中使用的是 `__atomic_fetch_add()`，类似于 `i++`。与 `__sync` 不同，`__atomic` 系列函数需要提供 `memorder` 参数。

程序代码 `__atomic.c`：
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <stdatomic.h>

#define MAX_N 10000000
#define N_THREAD 2

int data_index = 0;
int data[MAX_N];

void * func_thread_1(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        int temp = __atomic_fetch_add(&data_index, 1, memory_order_relaxed);
        data[temp] = i;        // even
    }
    return NULL;
}

void * func_thread_2(void * arg) {
    for (int i = 0; i < MAX_N; i += 2) {
        int temp = __atomic_fetch_add(&data_index, 1, memory_order_relaxed);
        data[temp] = i + 1;    // odd
    }
    return NULL;
}

int main() {
    pthread_t thread_1, thread_2;
    pthread_create(&thread_1, NULL, func_thread_1, NULL);
    pthread_create(&thread_2, NULL, func_thread_2, NULL);
    pthread_join(thread_1, NULL);
    pthread_join(thread_2, NULL);

    int max_delta = 0;
    for (int i = 1; i < MAX_N; i++) {
        int delta = abs(data[i] - data[i - 1]);
        if (delta > max_delta) {
            max_delta = delta;
        }
    }

    printf("Max delta = %d\n", max_delta);

    return 0;
}
```

#### 运行时间
运行时间如表所示：

| 程序       | `real`   | `user`   | `sys`    |
| :--------- | :------- | :------- | :------- |
| `peterson` | `0.073s` | `0.105s` | `0.000s` |
| `mutex`    | `0.087s` | `0.095s` | `0.036s` |
| `__sync`   | `0.200s` | `0.380s` | `0.000s` |
| `__atomic` | `0.305s` | `0.510s` | `0.010s` |

可以注意到，Peterson 的效率最好，Mutex 的效率其次，使用原子操作的效率较差。

若将 Peterson、Mutex 进入临界区进行 100 次操作换为 1 次操作，则效率变得比原子操作差很多。

### 参考资料
* (En) Peterson's algorithm - Wikipedia - https://en.wikipedia.org/wiki/Peterson%27s_algorithm
* (Zh) Peterson算法 - 维基百科 - https://zh.wikipedia.org/wiki/Peterson%E7%AE%97%E6%B3%95
* (En) Common threads: POSIX threads explained
  * Part 1 - https://www.ibm.com/developerworks/library/l-posix1/index.html
  * Part 2 - https://www.ibm.com/developerworks/library/l-posix2/index.html
  * Part 3 - https://www.ibm.com/developerworks/library/l-posix3/index.html （Part 3 并没有用到）
* (Zh) 通用线程：POSIX 线程详解
  * Part 1 - https://www.ibm.com/developerworks/cn/linux/thread/posix_thread1/index.html
  * Part 2 - https://www.ibm.com/developerworks/cn/linux/thread/posix_thread2/index.html
  * Part 3 - https://www.ibm.com/developerworks/cn/linux/thread/posix_thread3/index.html （Part 3 并没有用到）
* (Zh) 多线程下变量-原子操作 __sync_fetch_and_add等等 - https://blog.csdn.net/i_am_jojo/article/details/7591743
* (En) Multithreaded simple data type access and atomic variables - http://www.alexonlinux.com/multithreaded-simple-data-type-access-and-atomic-variables
* (En) Legacy __sync Built-in Functions for Atomic Memory Access - GCC 9.3 Manual - https://gcc.gnu.org/onlinedocs/gcc-9.3.0/gcc/_005f_005fsync-Builtins.html
* (En) Built-in Functions for Memory Model Aware Atomic Operations - GCC 9.3 Manual - https://gcc.gnu.org/onlinedocs/gcc-9.3.0/gcc/_005f_005fsync-Builtins.html

<style>
a {
    color: black;
}
</style>