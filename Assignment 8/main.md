# 操作系统 作业 8

## 1
设有两个优先级相同的进程 `P1`，`P2` 如下。令信号 `S1`，`S2` 的初值为 `0`，已知 `z = 2`，试问 `P1`，`P2` 并发运行结束后 `x`、`y`、`z` 的值？

```plain
    Process P1          Process P2
1   y := 1;             x := 1;
2   y := y + 2;         x := x + 1;
3   V(S1);              P(S1);
4   z := y + 1;         x := x + y;
5   P(S2);              V(S2);
6   y := z + y;         z := x + z;
```

### 解
* 根据信号 `S1` 和 `S2`，两个进程的运行顺序有下列要求：
  * `P2` 在第 3 行要等到 `P1` 执行完第 3 行后才能继续执行；
  * `P1` 在第 5 行要等到 `P2` 执行完第 5 行后才能继续执行；
* 以 `P1` 的第 3 行执行和 `P2` 的第 5 行执行两个时间点，将进程执行过程分为 A、B、C 三段：
  * `P1` 的第 1、2 行一定在 A 段执行，`P2` 的第 1、2 行可能在 A 或 B 段执行；
  * `P1` 的第 4 行可能在 B 或 C 段执行，`P2` 的第 3、4 行一定在 B 段执行；
  * `P1` 的第 6 行一定在 C 段执行，`P2` 的第 6 行一定在 C 段执行。
* 观察语句间的相关性：
  * `P1` 的第 1、2 行与 `P2` 的第 1、2 行之间不相关，A 段的代码执行顺序**不会**影响结果；
  * `P1` 的第 4 行与 `P2` 的第 1、2、3、4 行之间不相关，B 段的代码执行顺序**不会**影响结果。
  * `P1` 的第 4、6 行与 `P2` 的第 6 行之间存在相关性，C 段的代码执行顺序**会**影响结果。
* 在完成 A、B 两段一定能够完成的指令后（`P1` 的 1、2、3 行，`P2` 的 1、2、3、4、5 hang ），有 `x = 5`、`y = 3`、`z = 2`。
* `P1` 的第 4、6 行与 `P2` 的第 6 行，有 3 种完成顺序：
  * `P1` 的第 4 行，`P1` 的第 6 行，`P2` 的第 6 行：`x = 5`、`y = 7`、`z = 9`；
  * `P1` 的第 4 行，`P2` 的第 6 行，`P1` 的第 6 行：`x = 5`、`y = 12`、`z = 9`；
  * `P2` 的第 6 行，`P1` 的第 4 行，`P1` 的第 6 行：`x = 5`、`y = 7`、`z = 4`。

综上，并发运行结束后，`x`、`y`、`z` 的值共有 3 种可能：
* `x = 5`、`y = 7`、`z = 9`；
* `x = 5`、`y = 12`、`z = 9`；
* `x = 5`、`y = 7`、`z = 4`。


## 2
银行有 `n` 个柜员，每个顾客进入银行后先取一个号，并且等着叫号，当一个柜员空闲后，就叫下一个号。

请用 `PV` 操作分别实现：
* 顾客取号操作 `Customer_Service`
* 柜员服务操作 `Teller_Service`

### 解

C 风格的伪代码如下：
```c
int number_front = 0;       // 等待队列的队首号码
int number_back = 0;        // 等待队列的队尾号码

int signal_waiting = n;             // 信号量，等待的顾客数

int Customer_Service() {
    // 增加等待的顾客数
    D(signal_waiting);
    // 返回编号
    return number_back ++;
}

int Teller_Service() {
    while (1) {
        // 等待有顾客取号
        V(signal_waiting);
        // 叫号
        calling_number(number_front ++);
        // 服务
        do_service();
    }
}
```

## 3
多个线程的规约（Reduce）操作是把每个线程的结果按照某种运算（符合交换律和结合律）两两合并直到得到最终结果的过程。

试设计管程 `monitor` 实现一个 `8` 线程规约的过程，随机初始化 `16` 个整数，每个线程通过调用 `monitor.getTask` 获得 `2` 个数，相加后，返回一个数 `monitor.putResult`，然后再 `getTask()` 直到全部完成退出，最后打印归约过程和结果。

**要求**：为了模拟不均衡性，每个加法操作要加上随机的时间扰动，变动区间 `1~10 ms`。

**提示**：使用 `pthread_` 系列的 `cond_wait`、`cond_signal`、`mutex` 实现管程，使用 `rand()` 函数产生随机数，和随机执行时间。

### 解

程序采用 C 语言，代码如下：
```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <syscall.h>
#include <stdbool.h>
#include <sys/time.h>

#define N_TASK 16
#define ONE_MS 650000
#define N_THREAD 8

typedef struct {
    int a, b;
} Pair;

typedef struct Node {
    int num;
    struct Node * next;
} Node;

typedef struct {
    int cnt;
    Node * head;
    Node * nail;
} Queue;

void Queue_init(Queue * que) {
    que->cnt = 0;
    que->head = NULL;
    que->nail = NULL;
}

void Queue_push(Queue * que, int num) {
    Node * new_node = malloc(sizeof(Node));
    new_node->num = num;

    if (que->head == NULL) {
        que->head = new_node;
        que->nail = new_node;
        new_node->next = NULL;
    } else {
        que->nail->next = new_node;
        que->nail = new_node;
    }
    que->cnt += 1;
}

int Queue_pop(Queue * que) {
    Node * old_node = que->head;
    int res = old_node->num;
    if (old_node->next == NULL) {
        que->head = NULL;
        que->nail = NULL;
    } else {
        que->head = old_node->next;
    }
    que->cnt -= 1;
    free(old_node);
    return res;
}

int Queue_print(Queue * que) {
    for (Node * p = que->head; p != NULL; p = p->next) {
        printf("%d ", p->num);
    }
    printf("\n");
}

typedef struct {
    Queue que;
    pthread_mutex_t lock;
    pthread_cond_t can_reduce;
    int remind_task;
} Monitor;

void Monitor_init(Monitor * p) {
    p->remind_task = N_TASK - 1;
    Queue_init(&p->que);
    for (int i = 0; i < N_TASK; i++) {
        Queue_push(&p->que, rand() % 100);
    }

    pthread_mutex_init(&p->lock, NULL);
}

bool Monitor_getTask(Monitor * p, Pair * result) {
    pthread_mutex_lock(&p->lock);

    if (p->remind_task <= 0) {
        pthread_mutex_unlock(&p->lock);
        return false;
    }

    p->remind_task -= 1;

    while (p->que.cnt < 2) {
        pthread_cond_wait(&p->can_reduce, &p->lock);
    }

    result->a = Queue_pop(&p->que);
    result->b = Queue_pop(&p->que);

    printf("[getTask]   get %d, %d\n", result->a, result->b);

    pthread_mutex_unlock(&p->lock);
    return true;
}

void Monitor_putResult(Monitor * p, int res) {
    pthread_mutex_lock(&p->lock);

    Queue_push(&p->que, res);
    printf("[putResult] put %d\n", res);

    pthread_cond_signal(&p->can_reduce);

    pthread_mutex_unlock(&p->lock);
}

volatile void delay_ms(unsigned time) {
    // struct timeval t_start, t_end;
    // gettimeofday(&t_start, NULL);
    unsigned cnt = 0;
    for (unsigned i = 0; i < time; i++) {
        unsigned j = ONE_MS;
        while (j--) {
            cnt += i;
        }
    }
    // gettimeofday(&t_end, NULL);
    // printf("time: %ld", t_end.tv_usec - t_start.tv_usec);
}

Monitor task_monitor;

void * func_thread_reduce(void * arg) {
    Pair task;
    while (1) {
        if (!Monitor_getTask(&task_monitor, &task)) {
            printf("[Exit]\n");
            return NULL;
        }
        delay_ms(rand() % 10);
        int res = task.a + task.b;
        Monitor_putResult(&task_monitor, res);
    }
    return NULL;
}

int main() {
    Monitor_init(&task_monitor);
    printf("Start:\n");
    Queue_print(&task_monitor.que);

    pthread_t thread[N_THREAD];
    for (int i = 0; i < N_THREAD; i++) {
        pthread_create(thread + i, NULL, func_thread_reduce, NULL);
    }
    for (int i = 0; i < N_THREAD; i++) {
        pthread_join(thread[i], NULL);
    }

    printf("End:\n");
    Queue_print(&task_monitor.que);

    return 0;
}
```

程序运行结果：
```plain
Start:
83 86 77 15 93 35 86 92 49 21 62 27 90 59 63 26 
[getTask]   get 83, 86
[putResult] put 169
[getTask]   get 77, 15
[getTask]   get 93, 35
[getTask]   get 86, 92
[getTask]   get 49, 21
[getTask]   get 62, 27
[getTask]   get 90, 59
[putResult] put 92
[getTask]   get 63, 26
[putResult] put 128
[getTask]   get 169, 92
[putResult] put 70
[getTask]   get 128, 70
[putResult] put 198
[putResult] put 261
[getTask]   get 198, 261
[putResult] put 459
[putResult] put 178
[Exit]
[getTask]   get 459, 178
[putResult] put 149
[Exit]
[putResult] put 89
[Exit]
[getTask]   get 149, 89
[putResult] put 637
[Exit]
[putResult] put 89
[Exit]
[getTask]   get 637, 89
[putResult] put 238
[Exit]
[putResult] put 726
[Exit]
[getTask]   get 238, 726
[putResult] put 964
[Exit]
End:
964 
```

程序运行符合预期。