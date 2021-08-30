# 操作系统 作业 12

## 1
一个 RAID-5，有 5 个磁盘，每条含一个块（4 KB），采用如下图所示的映射。

| Disk 0 | Disk 1 | Disk 2 | Disk 3 | Disk 4 |
| :----: | :----: | :----: | :----: | :----: |
|   0    |   1    |   2    |   3    | **P0** |
|   5    |   6    |   7    | **P1** |   4    |
|   10   |   11   | **P2** |   8    |   9    |
|   15   | **P3** |   12   |   13   |   14   |
| **P4** |   16   |   17   |   18   |   19   |

如果每个磁盘的平均寻道时间是 4 ms，旋转速度是 7200 RPM（即每分钟 7200 转），请问：
1. 从这个 RAID-5 中读出一个数据块的时间是多少？
2. 向这个 RAID-5 中写入一个数据块的时间是多少？
3. 向这个 RAID-5 中写入两个连续的数据块的时间是多少？（提示：需分别考虑两个连续数据块位于同一条带和位于不同条带的两种情况）

### 解

1. 单块硬盘的平均寻道时间是 4 ms，平均旋转延迟是 0.5 ÷ (7200 / 60000) ms ≈ 4.17 ms，4 KB 的数据传输时间可以忽略。因此，读出一个数据块的时间是 4 ms + 4.17 ms ≈ 8.17 ms。
   
2. RAID-5 写入一个数据块需要先读出旧块（旧块 + 旧校验块），然后再写入新块（新块 + 新校验块），读和写过程均是两个块并行的。读出旧块和写入新块各需要 8.17 ms，故写入一个数据块总共需要 2 × 8.17 ms ≈ 16.3 ms。
   
3. 有如下两种情况：
   * 若两个连续块位于同一个条带，那么其写入过程等同于写入一个数据块，需要 16.3 ms。
   * 若两个连续块位于两个条带上，那么根据映射表，第一块一定与第二块对应的校验块在同一块硬盘上，第二块一定与第一块对应的校验块在同一块硬盘上。因此，对两个连续数据块的写入无法并行，总共需要 2 × 16.3 ms ≈ 32.7 ms。

## 3
一个 UNIX 文件系统的文件块索引采用多级间址，10 个直接指针，1 个一级间址指针，1 个二级间址指针，1 个三级间址指针。假设块大小为 4 KB（4096 字节），磁盘块地址为 4 个字节。
1. 请问该索引结构能够索引的最大文件是多大？
2. 请问一个 1 GB 的文件需要几级间址？它总共有多少间址块？其中各级间址块分别是多少？
3. 一个 10 GB 的文件需要几级间址？它总共有多少间址块？其中各级间址块分别是多少？如何找到第 200,000 块？

### 解

1. * 每个块可以存储 4096 / 4 = 1024 个块地址；
   * 该索引结构支持 10 + 1024 + 1024<sup>2</sup> + 1024<sup>3</sup> = 1,074,791,434 个文件块；
   * 故支持最大 4 × 1,074,791,434 KB = 4,299,165,736 KB ≈ 4,198,404 MB ≈ 4,100 GB 的文件。
  
2. * 1 GB 的文件需要 1024 × 1024 ÷ 4 = 262,144 个文件块。
   * 采用一级间址支持 10 + 1024 = 1034 个文件块，不够使用；
   * 采用二级间址支持 10 + 1024 + 1024<sup>2</sup> = 1,049,610 个文件块，足够使用，因此需要二级间址；
   * 文件块中，有 10 个为直接索引的文件块，有 1024 个采用一级间址的文件块，有 262,144 - 10 - 1024 = 261,110 个采用二级间址的文件块；
   * 一级和二级间址总共用了 2 个一级间址块，二级间址用了 ⌈261,110 ÷ 1024⌉ = 255 个间址块（向上取整），总共使用了 257 个间址块。
  
3. * 10 GB 的文件需要 10 × 1024 × 1024 ÷ 4 = 2,621,440 个文件块；
   * 根据 2 的计算，采用二级间址不够使用，需要采用三级间址；
   * 文件块中，有 10 个为直接索引的文件块，有 1024 个采用一级间址的文件块，有 1,048,576 个采用二级间址的文件块，有 2,621,440 - 1,048,576 - 1024 - 10 = 1,571,830 个采用三级间址的文件块；
   * 一级、二级和三级间址总共使用了 3 个一级间址块；二级间址使用了 1024 个二级间址块，三级间址使用了 ⌈1,571,830 ÷ (1024 × 1024)⌉ = 2 个二级间址块，总共使用了 1024 + 2 = 1026 个二级间址块；三级间址使用了 1024 + ⌈(1,571,830 - 1024 × 1024) ÷ 1024⌉ = 1535 个三级间址块。
   * 总共使用了 3 + 1026 + 1535 = 2,564 个间址块。
   * 显然，第 200,000 块在二级间址中，是二级间址部分的第 200,000 - 10 - 1024 = 198,966 块。那么，它是二级间址中第 ⌊198,966 ÷ 1024⌋ = 194 块二级间址块（从 0 开始计）的第 198,966 mod 1024 = 310 块。

## 4
A certain file system uses 4-KB disk blocks. The median file size is 1 KB. If all files were exactly 1 KB, what fraction of the disk space would be wasted? Do you think the wastage for a real file system will be higher than this number of lower than it? Explain your answer.

### Solution
An 1-KB file occupies a 4-KB disk block, with 3 KB space wasted. Hence 75% of the disk space would be wasted. 

I think the wastage for a real file system will be lower than this number. Because a real file system not only contains small files, but also contains large files. The space efficiency of large files is much better than small files.

## 5
One way to use contiguous allocation of the disk and not suffer from holes is to compact the disk every time a file is removed. Since all files are contiguous, copying file requires a seek and rotational delay to read the file, followed by the transfer at full speed. Writing the file back requires the same work. Assuming a seek time of 5 msec, a rotational delay of 4 msec, a transfer rate of 80 MB/sec, and an average file of 8 KB, how long does it take to read a file into main memory and then write it back to the disk at a new location? Using these numbers, how long would it take to compact half of a 16-GB disk?

### Solution

* It take 5 msec + 4 msec + 8 / (80 * 1024 / 1000) msec ≈ 9.098 msec to read a file into main memory. It takes another 9.098 msec to write the file back to the disk. Hence it takes 18.195 msec in total.
* Half of 16-GB disk contains 8 * 1024 * 1024 / 8 = 1,048,576 ≈ 1 × 10<sup>6</sup> files. As moving one file takes 18.195 msec, it will take 18.195 × 1,048,576 msec ≈ 19,078.8 sec ≈ 5.3 hour.