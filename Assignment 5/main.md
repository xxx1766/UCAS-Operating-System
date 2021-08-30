# 操作系统 作业 5

## 1.
Five jobs are waiting to be run. Their expected run times are $9$, $6$, $3$, $5$, and $X$.
In what order should they be run to minimize average response time? (Your answer will depend on $X$.)

**Solution**

Using STCF algorithm can minimize average response time.

* If $X \leq 3$, The order should be $X$, $3$, $5$, $6$, $9$.   
  The average response time is $[0 + X + (X + 3) + (X + 3 + 5) + (X + 3 + 5 + 6)] / 5 = 0.8 X + 5$
  * If $X = 1$, The average response time is $5.8$
  * If $X = 2$, The average response time is $6.6$
  * If $X = 3$, The average response time is $7.4$
* If $3 < X \leq 5$, The order should be $3$, $X$, $5$, $6$, $9$.   
  The average response time is $[0 + 3 + (3 + X) + (3 + X + 5) + (3 + X + 5 + 6)] / 5 = 0.6 X + 5.6$
  * If $X = 4$, The average response time is $8$
  * If $X = 5$, The average response time is $8.6$
* If $5 < X \leq 6$, The order should be $3$, $5$, $X$, $6$, $9$.   
  The average response time is $[0 + 3 + (3 + 5) + (3 + 5 + X) + (3 + 5 + X + 6)] / 5 = 0.4 X + 6.6$
  * If $X = 6$, The average response time is $9$
* If $6 < X \leq 9$, The order should be $3$, $5$, $6$, $X$, $9$.   
  The average response time is $[0 + 3 + (3 + 5) + (3 + 5 + 6) + (3 + 5 + 6 + X)] / 5 = 0.2 X + 7.8$
  * If $X = 7$, The average response time is $9.2$
  * If $X = 8$, The average response time is $9.4$
  * If $X = 9$, The average response time is $9.6$
* If $X > 9$, The order should be $3$, $5$, $6$, $9$, $X$.   
  The average response time is $[0 + 3 + (3 + 5) + (3 + 5 + 6) + (3 + 5 + 6 + 9)] / 5 = 9.6$

## 2. 
Five batch jobs $A$ through $E$, arrive at a computer center at almost the same 
time. They have estimated running times of $10$, $6$, $2$, $4$, and $8$ minutes. Their
(externally determined) priorities are $3$, $5$, $2$, $1$, and $4$, respectively, with $5$ being
the highest priority. For each of the following scheduling algorithms, determine
the mean process turn-around time. Ignore process switching overhead.

(a) Round robin.  
(b) Priority scheduling  
(c) First-come, first-served (run in order $10$, $6$, $2$, $4$, $8$).  
(d) Shortest job first.  

For (a), assume that the system is multiprogrammed, and that each job gets its
fair share of the CPU. For (b) through (d) assume that only one job at a time
runs, until it finishes. All jobs are completely CPU bound.

**Solution**

(a) Assume that the time quantum is 1 minute.

The processing is below.
```
Task            A   B   C   D   E
Estimated Time  10  6   2   4   8 
Run     (1)     1   2   3   4   5
        (2)     6   7   8*  9   10
        (3)     11  12      13  14
        (4)     15  16      17* 18
        (5)     19  20          21
        (6)     22  23*         24
        (7)     25              26
        (8)     27              28*
        (9)     29
        (10)    30*
```

So average process turn-around time is $(8 + 17 + 23 + 28 + 30) / 5 = 21.2$ minutes.

(b)

The processing is below.
```
Task            B   E   A   C   D
Priority        5   4   3   2   1
Estimated Time  6   8   10  2   4
Run Finish At   6   14  24  26  30
```

So average process turn-around time is $(6 + 14 + 24 + 26 + 30) / 5 = 20$ minutes.

(c)

The processing is below.
```
Task            A   B   C   D   E
Estimated Time  10  6   2   4   8 
Run Finish At   10  16  18  22  30
```

So average process turn-around time is $(10 + 16 + 18 + 22 + 30) / 5 = 19.2$ minutes.

(d)

The processing is below.
```
Task            C   D   B   E   A
Estimated Time  2   4   6   8   10
Run Finish At   2   6   12  20  30
```

So average process turn-around time is $(2 + 6 + 12 + 20 + 30) / 5 = 14$ minutes.

## 3.
A real-time system needs to handle two voice calls that each run every $5$
msec and consume $1$ msec of CPU time per burst, plus one video at $24$
frames/sec, with each frame requiring $20$ msec of CPU time. Is this system
schedulable?

**Solution**

As
$$
\sum \frac{C_i}{T_i} = \frac{1\ \text{msec}}{5\ \text{msec}} + \frac{1\ \text{msec}}{5\ \text{msec}} + \frac{20\ \text{msec} \times 24}{1000\ \text{msec}} = \frac{22}{25} < 1
$$

This system is schedulable.