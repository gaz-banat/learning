

# CONCEPTS


## CPU

us      -   User Time

sy      -   system time

id

wa      -   io wait

st      -   steal time, waiting for cpu from hypervisor


irq

softirq

nice





## MEMORY




## IO




## NETWORK






# TOOLS


## TOP



## VMSTAT

1. r value should not be higher than number of cpu's on the system

2. si and so values should not be non zero together consistently

3. si and so values should be generally less than 10

4. cs value is high could mean that too much process switching (context switching) is occurring

5. cs is higher than sy could mean that system is doing more context switching than actual work




6. High r with high cs - possible lock contention

Lock contention occurs whenever one process or thread attempts to acquire a lock held by another process or thread. 
The more granular the available locks, the less likely one process/thread will request a lock held by the other. 
For example, locking a row rather than the entire table, or locking a cell rather than the entire row.

When you are seeing blocked processes or high values on waiting on I/O (wa), it usually signifies either 
- real I/O issues where you are waiting for file accesses or 
- an I/O condition associated with paging due to a lack of memory on your system.

7. if us+sy is always greater than 80% - then CPU is approaching its limits

8. if us+sy = 100%   -> possible CPU bottleneck

9. if sy is high - your application is issuing many system calls to the kernel and asking the kernel to work. It measures how heavily the appl. is using kernel services.\


10. if sy is higher than us - this means your system is spending less time on real work (not good)



## FREE



## PIDSTAT



## DMESG