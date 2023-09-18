# mysql57-write-types-ratio-experiment


2023 Fall SKKU Database Project

## Overview
This week, you will quantify the ratio of three disk write types in MySQL(single page flush, flush list flush, LRU list flush). You will first add some codes to print the flush types. After running TPC-C with modified MySQL source code, calcualte the ratio of three flush types in MySQL varying buffer pool size. (10%, 30%, 50%)

Follow the guide below. If you have any questions, contact me via email. (kyongshikl@gmail.com)


## Instructions


1. Restart a MySQL server
- Before starting a MySQL server, update the buffer pool size to 10% (then, 20%, 30%, 40%, 50%) of your TPC-C database size. For example, if you load 20 warehouses (e.g., about 2G database size), change the value of innodb_buffer_pool_size in my.cnf to 200M:
```bash
$ vi /path/to/my.cnf
...
innodb_buffer_pool_size=200M
...
```

- Start a MySQL server:
```bash
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

2. Run the TPC-C benchmark
Run the benchmark by modifying the experimental parameters to match your system specifications. For example:
```bash
$ ./tpcc_start -h 127.0.0.1 -S /tmp/mysql.sock -d tpcc -u root -p "yourPassword" -w 20 -c 8 -r 10 -l 1200 | tee tpcc-result.txt
```

3. Monitor the buffer hit/miss ratio of MySQL
- While running the benchmark, collect performance metrics (e.g., I/O status, transaction throughput, hit/miss ratio) and record them in a separate file for future analysis. Refer to the [performance monitoring guide](https://github.com/LeeBohyun/SWE3033-S2023/blob/main/week1/reference/performance-analysis-guide.md).
- Also, regarding hit ratio, this is a way to monitor buffer hit rate.
```bash
$ ./bin/mysql -uroot -pyourPassword
Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 8Server version: 8.0.15 Source distribution
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show engine innodb status;
...
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 2353004544
Dictionary memory allocated 373557
Buffer pool size   524288
Free buffers       0
Database pages     524287
Old database pages 193695
Modified db pages  0
Pending reads      1
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 36985
0.00 youngs/s, 2465.50 non-youngs/s
Pages read 576950, created 160, written 177
38431.37 reads/s, 0.13 creates/s, 11.13 writes/s
Buffer pool hit rate 986 / 1000, young-making rate 0 / 1000 not 63 / 1000
Pages read ahead 0.00/s, evicted without access 3444.10/s, Random read ahead 0.00/s
LRU len: 524287, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
...
```
- Note the ``Buffer pool hit rate`` metric
- It means the buffer pool page hit rate for pages read from the buffer pool vs. from disk storage
- In the example above, ``buffer pool hit rate = 0.986``


4. After the benchmark ends, shut down the MySQL server:
```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
or
$ sudo killall mysqld
```

## Reference
- https://www.percona.com/blog/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/
- https://github.com/Percona-Lab/tpcc-mysql
