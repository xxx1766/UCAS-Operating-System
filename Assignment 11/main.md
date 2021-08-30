# 操作系统 作业 11

## 1
一个磁盘的平均寻道时间是 $4$ ms，旋转速度是 $7200$ RPM（即每分钟 $7200$ 转），它的每条磁道有 $500$ 个扇区，每个扇区 $512$ 字节。
1. 请问它的最大数据传输速率是多少字节/秒？
2. 读一个扇区的平均时间是多少毫秒（ms）？
3. 如果它的密度增加一倍，即每条磁道有 $1000$ 个扇区，每个扇区依然是 $512$ 字节，请问它的最大数据传输速率是多少字节/秒？读一个扇区的平均时间是多少毫秒（ms）？

### Solution
1. 它的最大传输速率为
   $$
   500 \times 512 \text{ B} \times \frac{7200}{60} \text{ s}^{-1} = 30,720,000 \text{ B/s} = 30,000 \text{ KB/s}
   $$
   
2. 读一个扇区的平均时间为
   $$
   4 \text{ ms} + 0.5 \div \frac{7200}{60000} \text{ ms} + 512 \text{ B} \div 30720 \text{ B/ms} = 8.18 \text{ ms}
   $$
3. 如果密度增加一倍，
   * 它的最大传输速率为 
     $$
     1000 \times 512 \text{ B} \times \frac{7200}{60} \text{ s}^{-1} = 61,440,000 \text{ B/s} = 60,000 \text{ KB/s}
     $$
   * 读一个扇区的平均时间为
     $$
     4 \text{ ms} + 0.5 \div \frac{7200}{60000} \text{ ms} + 512 \text{ B} \div 61440 \text{ B/ms} = 8.175 \text{ ms}
     $$
   可以注意到，最大传输速率是原来的两倍，但读取一个扇区的平均时间基本没有变化。


## 2
Consider a magnetic disk consisting of 16 heads and 400 cylinders. This disk has four 100 cylinder zones with the cylinders in different zones containing 160, 200, 240 and 280 sectors, respectively. Assume that each sector contains 512 bytes, average seek time between adjacent cylinders is 1 msec, and the disk rotates at 7200 RPM. Calculate the (a) disk capacity, (b) optimal track skew, and (c) maximum data transfer rate.

### Solution

**(a)** The disk capacity is 
$$
16 \times 100 \times (160 + 200 + 240 + 280) \times 512 \text{ B} = 720,896,000 \text{ B} = 687.5 \text{ MB}
$$

**(b)** During the average seek time of 1 msec, the disk rotates $0.001 \times 7200 / 60 = 0.12$ revolutions. Hence,
* Zone 1: The head passes $0.12 \times 160 = 19.2$ sectors, so the optimal track skew is 19.2 sectors.
* Zone 2: The head passes $0.12 \times 200 = 24$ sectors, so the optimal track skew is 24 sectors.
* Zone 3: The head passes $0.12 \times 240 = 28.8$ sectors, so the optimal track skew is 28.8 sectors.
* Zone 4: The head passes $0.12 \times 280 = 33.6$ sectors, so the optimal track skew is 33.6 sectors.

**(c)** The maximum data transfer rate is
$$
280 \times 512 \text{ B} \times \frac{7200}{60} \text{ s}^{-1} = 17,203,100 \text{ B/s} = 16,800 \text{ KB/s}
$$

## 3
Disk requests come in to the disk driver for cylinders 10, 22, 20, 2, 40, 6, and 38, in that order. A seek takes 6 msec per cylinder. How much seek time is needed for
* (a) First-come, first served.
* (b) Closest cylinder next.
* (c) Elevator algorithm (initially moving upward).
In all cases, the arm is initially at cylinder 20.

### Solution
#### (a) FIFO
| Current Cylinder | Seek time (msec) | Total Time (msec) |
| :--------------: | :--------------: | :---------------: |
|        20        |                  |         0         |
|        10        |   6 × 10 = 60    |        60         |
|        22        |   6 × 12 = 72    |        132        |
|        20        |    6 × 2 = 12    |        144        |
|        2         |   6 × 18 = 108   |        252        |
|        40        |   6 × 38 = 228   |        480        |
|        6         |   6 × 34 = 204   |        684        |
|        38        |   6 × 32 = 192   |        876        |

#### (b) Closest cylinder next
| Current Cylinder | Seek time (msec) | Total Time (msec) |
| :--------------: | :--------------: | :---------------: |
|        20        |                  |         0         |
|        20        |        0         |         0         |
|        22        |    6 × 2 = 12    |        12         |
|        10        |   6 × 12 = 72    |        84         |
|        6         |    6 × 4 = 24    |        108        |
|        2         |    6 × 4 = 24    |        132        |
|        38        |   6 × 36 = 216   |        348        |
|        40        |    6 × 2 = 12    |        360        |

#### (c) Elevator algorithm
| Current Cylinder | Seek time (msec) | Total Time (msec) |
| :--------------: | :--------------: | :---------------: |
|        20        |                  |         0         |
|        20        |        0         |         0         |
|        22        |    6 × 2 = 12    |        12         |
|        38        |   6 × 16 = 96    |        108        |
|        40        |    6 × 2 = 12    |        120        |
|        10        |   6 × 30 = 180   |        300        |
|        6         |    6 × 4 = 24    |        324        |
|        2         |    6 × 4 = 24    |        348        |


## 4
A personal computer salesman visiting a university in South-West Amsterdam remarked during his sales pitch that his company had devoted substantial effort to making their version of UNIX very fast. As an example, he noted that their disk driver used the elevator algorithm and also queued multiple requests within a cylinder in sector order. A student, Harry Hacker, was impressed ant bought one. He tool it home and wrote a program to randomly read 10,000 blocks spread across the disk. To his amazement, the performance that he measured was identical to what would be expected from first-come, first-served. Was the salesman lying?

### Solution
The salesman was not lying.

The program written by Harry Hacker is not a multithreaded program, so all accesses to disk are synchronous. The program will not continue until the last reading result returns. In other words, there is only one reading request at any time. Therefore, it is first-come, first-served.

## 5
假设磁盘的平均寻道时间是 6 ms，旋转速率是 15,000 RPM（即每分钟 15,000 转），每条磁道 1 MB。请计算大小分别为 512 B、1 KB 和 4 KB 的数据块的传输带宽。

### Solution

* 最大传输速率：
  $$
  1 \text{ MB} \times \frac{15000}{60} \text{ s}^{-1} = 250 \text{ MB/s} = 256,000 \text{ KB/s}
  $$
* 平均旋转时间：
  $$ 0.5 \div \frac{15000}{60} = 0.002 \text{ s} $$
  <div style="page-break-after:always;">
* 数据块的传输带宽：
  * 512 B：
    $$
    \frac{0.5 \text{ KB}}{0.002 \text{ s} + 0.006 \text{ s} + \frac{0.5 \text{ KB}}{256000 \text{ KB/s}}} \approx 62.48 \text{ KB/s}
    $$
  * 1 KB：
    $$
    \frac{1 \text{ KB}}{0.002 \text{ s} + 0.006 \text{ s} + \frac{1 \text{ KB}}{256000 \text{ KB/s}}} \approx 124.94 \text{ KB/s}
    $$
  * 4 KB：
    $$
    \frac{4 \text{ KB}}{0.002 \text{ s} + 0.006 \text{ s} + \frac{4 \text{ KB}}{256000 \text{ KB/s}}} \approx 499.03 \text{ KB/s}
    $$