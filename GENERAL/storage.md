

# CONCEPTS

## IOPS

input       = write
output      = read
operation   = transaction
per second is the key here

Industry standard for IOPS on a device is 4KB per operation/transaction.


## Block size

NOTE: K is KB

Database operations could be 4K to 8K per block
Backup could be running at 256K per block
Video transfer could be at 512K per block


Generally speaking higher the block size lower the IOPS



## Latency

Time taken to complete an IO, measured in ms
1000ms = 1s



## Throughput/Bandwidth

This is about "Block Size x IOPS"

A storage system transferring 4KB blocks at 10000 IOPS will give throughput of 40MB/s
4KB x 10000 IOPS  =  40000KB/s = 40MB/s

A storage system transferring 1MB blocks at 250 IOPS will give a throughput of 250MB/s
1MB x 250 IOPS = 250MB/s 




## WORKLOADS

High IOPS workloads/Many small random reads and writes
 - Database Systems
 - Virtualization
 - Transaction processing

High Througput workloads/large sequential access
 - video transfer
 - large file upload, download
 - backups
