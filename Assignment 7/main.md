# 操作系统 作业 7

## 1
A system has four processes and five allocatable resources. The current allocation and maximum needs are as follows:

|           | Allocated   | Maximum     | Available   |
| --------: | :---------- | :---------- | :---------- |
| Process A | `1 0 2 1 1` | `1 1 2 1 3` | `0 0 x 1 1` |
| Process B | `2 0 1 1 0` | `2 2 2 1 0` |             |
| Process C | `1 1 0 1 0` | `2 1 3 1 0` |             |
| Process D | `1 1 1 1 0` | `1 1 2 2 1` |             |

What is the smallest value of x for which this is a safe state?

### Solution

First, we can get the need matrix of resources:

|           | Allocated   | Maximum     | Needs       | Available   |
| --------: | :---------- | :---------- | :---------- | :---------- |
| Process A | `1 0 2 1 1` | `1 1 2 1 3` | `0 1 0 0 2` | `0 0 x 1 1` |
| Process B | `2 0 1 1 0` | `2 2 2 1 0` | `0 2 1 0 0` |             |
| Process C | `1 1 0 1 0` | `2 1 3 1 0` | `1 0 3 0 0` |             |
| Process D | `1 1 1 1 0` | `1 1 2 2 1` | `0 0 1 1 1` |             |

* If `x = 0`:
  * Then there are `0 0 0 1 1` instances available. None of processes can get enough resources and will execute.
  * Hence this is not a safe state.
* If `x = 1`:
  * Then there are `0 0 1 1 1` instances available. Process D can get enough resources and will execute.
  * Then there are `1 1 2 2 1` instances available. None of processes can get enough resources and will execute.
  * Hence this is not a safe state.
* If `x = 2`:
  * Then there are `0 0 2 1 1` instances available. Process D can get enough resources and will execute.
  * Then there are `1 1 3 2 1` instances available. Process C can get enough resources and will execute.
  * Then there are `2 2 3 3 1` instances available. Process B can get enough resources and will execute.
  * Then there are `4 2 4 4 1` instances available. However, Process A cannot get enough No.5 resources because there are only 2 No.5 resource in all.

Therefore, for any value of `x`, the system is always unsafe. If `2` is the smallest value of `x` for which this is safest.


## 2
The processes, A and B, each need three records, 1, 2, and 3, in a database. If
A asks for them in the order 1, 2, 3, and B asks for them in the same order,
deadlock is not possible. However, if B asks for them in the order 3, 2, 1, then
deadlock is possible. With three resources, there are 3! or six possible
combinations each process can request the resources. What fraction of all the
combinations is guaranteed to be deadlock free?

### Solution

If A and B ask a same record at first, only one of them will get it. That means the other will be blocked until the first one finished. Hence deadlock is not possible in this state.

If A and B ask two different records at first, they may each get one record. That means both of them will be blocked because both of them cannot get the record which the other one got. Hence deadlock is possible in this state.

Without loss of generality, we can assume A asks for records in the order 1, 2, 3. Then B have 6 possible ask order:

* 1, 2, 3
* 1, 3, 2
* 2, 1, 3
* 2, 3, 1
* 3, 1, 2
* 3, 2, 1

As mentioned earlier, there are only two combinations are guaranteed to be deadlock free (1, 2, 3 and 1, 3, 2).

Therefore, only 1/3  of all the combinations is guaranteed to be deadlock free.