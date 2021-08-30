# 操作系统 作业 10

## 1
If FIFO page replacement is used with four page frames and eight pages, how many page faults will occur with the reference string `0172327103` if the four frames are initially empty? Now repeat this problem for LRU.

### Solution

#### FIFO
```
Reference     0     1     7     2     3     2     7     1     0     3
Page frames   0___  01__  017_  0172  1723  1723  1723  1723  7230  7230
Page Fault    Yes   Yes   Yes   Yes   Yes   No    No    No    Yes   No
```

Therefore, 6 page faults will occur in FIFO page.

#### LRU
```
Reference     0     1     7     2     3     2     7     1     0     3
Page frames   0___  01__  017_  0172  1723  1732  1327  3271  2710  7103
Page Fault    Yes   Yes   Yes   Yes   Yes   No    No    No    Yes   Yes
```
Therefore, 7 page faults will occur in LRU page.

## 2
Suppose that the virtual page reference stream contains repetitions of long sequences of page references followed occasionally by a random page reference. For example, the sequence: 0, 1, ..., 511, 431, 0, 1, ..., 511, 332, 0, 1, ... consists of repetitions of the sequence 0, 1, ..., 511 followed by a random reference to pages 431 and 332.

Why will the standard replacement algorithms (LRU, FIFO, clock) not be effective in handling this workload for a page allocation that is less than the sequence length?

### Solution
在题目的条件下，对同一页的两次访问间隔很长，超过了页空间可容纳的最大数量，内存中的还未等到下一次访问，便会被换出内存，LRU、clock 基本已经退化为 FIFO。这意味着，每一次访问都会造成 Page Fault，大大降低了算法的效率。


## 3

A computer has four page frames. The time of loading, time of last access, and the $R$ and $M$ bits for each page are shown below (the times are in clock ticks):

| Page | Loaded | last ref. | $R$  | $M$  |
| :--- | :----- | :-------- | :--- | :--- |
| 0    | 126    | 280       | 1    | 0    |
| 1    | 230    | 265       | 0    | 1    |
| 2    | 140    | 270       | 0    | 0    |
| 3    | 110    | 285       | 1    | 1    |

1. Which page will NRU replace?
2. Which page will FIFO replace?
3. Which page will LRU replace?
4. Which page will second chance replace?

### Solution

1. Page 2 will NRU replace because both $R$ and $M$ bits are 0.
2. Page 3 will FIFO replace because it is the first page loaded into memory.
3. Page 1 will LRU replace because it is the least recently used page.
4. Page 2 will second chance replace because it is the first page loaded into memory with $R$ bits 0.

## 4
Consider the following two-dimensional array:
```c
    int X[64][64];
```

Suppose that a system has four page frames and each frame is 128 words (an integer occupies one word). Programs that manipulate the `X` array fit into exactly one page and always occupy page 0. The data are swapped in and out of the other three frames. The `X` array is stored in row-major order (i.e., `X[0][1]` follows `X[0][0]` in memory). Which of the two code fragments shown below will generate the lowest number of page faults? Explain and compute the total number of page faults.

*Fragment A*
```c
for (int j = 0; j < 64; j++)
    for (int i = 0; i < 64; i++)
        X[i][j] = 0;
```

*Fragment B*
```c
for (int i = 0; i < 64; i++)
    for (int j = 0; j < 64; j++)
        X[i][j] = 0;
```

### Solution
* The `X` array is stored in row-major order, and each frame is 128 words.
  * Hence every two rows of `X` array are in one page. For example, `X[0][0] ~ X[1][63]` are in one page.
* For *Fragment A*, the loop accesses `X` array by column, 
  * The inner loop accesses 32 pages sequentially, which causes 32 page faults.
  * The outer loop repeats such procure 64 times, so the program cause 32 × 64 = 2048 page faults.
* For *Fragment A*, the loop accesses `X` array by row.
  * The loop accesses `X` array in the order that `X` array is stored in memory. So the program access 32 pages one by one and causes 32 page faults.

Therefore, *Fragment B* will generate the lowest number of page faults.

## 5
We consider a program which has the two segments shown below consisting of instructions in segment 0, and read/write data in segment 1. Segment 0 has read/execute protection, and segment 1 has just read/write protection. The memory system is a demand-paged virtual memory system with virtual addresses that have a 4-bit page number, and a 10-bit offset. The page tables and protection are as follows (all numbers in the table are in decimal):

<table>
    <thead>
        <tr>
            <th colspan="2" style="text-align: center;">Segment 0</th>
            <th colspan="2" style="text-align: center;">Segment 1</th>
        </tr>
        <tr>
            <th colspan="2" style="text-align: center;">Read/Execute</th>
            <th colspan="2" style="text-align: center;">Read/Write</th>
        </tr>
        <tr>
            <th style="text-align: center;">Virtual Page #</th>
            <th style="text-align: center;">Page  frame #</th>
            <th style="text-align: center;">Virtual Page #</th>
            <th style="text-align: center;">Page frame #</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td style="text-align: center;">0</td>
            <td style="text-align: center;">2</td>
            <td style="text-align: center;">0</td>
            <td style="text-align: center;">On Disk</td>
        </tr>
        <tr>
            <td style="text-align: center;">1</td>
            <td style="text-align: center;">On Disk</td>
            <td style="text-align: center;">1</td>
            <td style="text-align: center;">14</td>
        </tr>
        <tr>
            <td style="text-align: center;">2</td>
            <td style="text-align: center;">11</td>
            <td style="text-align: center;">2</td>
            <td style="text-align: center;">9</td>
        </tr>
        <tr>
            <td style="text-align: center;">3</td>
            <td style="text-align: center;">5</td>
            <td style="text-align: center;">3</td>
            <td style="text-align: center;">6</td>
        </tr>
        <tr>
            <td style="text-align: center;">4</td>
            <td style="text-align: center;">On Disk</td>
            <td style="text-align: center;">4</td>
            <td style="text-align: center;">On Disk</td>
        </tr>
        <tr>
            <td style="text-align: center;">5</td>
            <td style="text-align: center;">On Disk</td>
            <td style="text-align: center;">5</td>
            <td style="text-align: center;">13</td>
        </tr>
        <tr>
            <td style="text-align: center;">6</td>
            <td style="text-align: center;">4</td>
            <td style="text-align: center;">6</td>
            <td style="text-align: center;">8</td>
        </tr>
        <tr>
            <td style="text-align: center;">7</td>
            <td style="text-align: center;">3</td>
            <td style="text-align: center;">7</td>
            <td style="text-align: center;">12</td>
        </tr>
    </tbody>
</table>

For each of the following cases, either give the real (actual) memory address which results from dynamic address translation or identify the type of fault which occurs (either page or protection fault).

1. Fetch from segment 1, page 1, offset 3
2. Store into segment 0, page 0, offset 16
3. Fetch from segment 1, page 4, offset 28
4. Jump to location in segment 1, page3, offset 32

### Solution

1. The real memory address is `0x3803`.
2. Protection fault occurs, because segment 0 is not writable.
3. Page fault occurs, because segment 1, page 4 is on disk.
4. Protection fault occurs, because segment 1 is note executable.