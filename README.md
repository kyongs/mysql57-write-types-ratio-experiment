# mysql57-write-types-ratio-experiment


2023 Fall SKKU Database Project

## Overview
This week, you will quantify the ratio of three disk write types in MySQL(single page flush, flush list flush, LRU list flush). You will first add some codes to print the flush types. After running TPC-C with modified MySQL source code, calcualte the ratio of three flush types in MySQL varying buffer pool size. (10%, 30%, 50%)

Follow the guide below. If you have any questions, contact me via email. (kyongshikl@gmail.com)


## Instructions
1. First, add the codes that log the flush types.
- You have to modify the  `mysql-5.7.33/storage/innobase/buf/buf0flu.cc`.
- `HINT`: Add the fprintf code like the below example (buf0flu.cc: buf_flush_single_page_from_LRU()).

```
/******************************************************************//**
This function picks up a single page from the tail of the LRU
list, flushes it (if it is dirty), removes it from page_hash and LRU
list and puts it on the free list. It is called from user threads when
they are unable to find a replaceable page at the tail of the LRU
list i.e.: when the background LRU flushing in the page_cleaner thread
is not fast enough to keep pace with the workload.
@return true if success. */
bool
buf_flush_single_page_from_LRU(
/*===========================*/
	buf_pool_t*	buf_pool)	/*!< in/out: buffer pool instance */
{
	ulint		scanned;
	buf_page_t*	bpage;
	ibool		freed;

	buf_pool_mutex_enter(buf_pool);
        fprintf(stderr, "single page flush\n"); //ADDED LINE
  ...
  
	return(freed);

}
```


2. Rebuild the MySQL.
```bash
$ cd /path/to/mysql-basedir
$ make -j install
```

3. Start a MySQL server.
  - Before starting a MySQL server, update the buffer pool size to 10% (then,30%, 50%) of your TPC-C database size in my.cnf.
  - For example, if you load 10 warehouses (e.g., about 1G database size), change the value of innodb_buffer_pool_size in my.cnf to 100M:

```bash
$ vi /path/to/my.cnf
...
innodb_buffer_pool_size=100M
...
```

```bash
$ ./bin/mysqld_safe --default-file=/path/to/my.cnf
```


4. Run the TPC-C benchmark
Run the benchmark by modifying the experimental parameters to match your system specifications. 
```bash
$ ./tpcc_start -h 127.0.0.1 -S /tmp/mysql.sock -d tpcc -u root -p "yourPassword" -w 10 -c 8 -r 10 -l 600 | tee tpcc-result.txt
```

Use the following tpc-c configuration:
- warehouse: 10
- connection: 8
- ramp-up time: 10 sec
- experiment time (-l): minimum 300 sec
  > You should be able to monitor single-page flush when the buffer pool size is 10%. If the single-page flush is not occurring, you need to extend the experiment time.

5. You can check if MySQL has been rebuilt sucessfully by looking at the `/path/to-test-data/mysql_error.log`.
```bash
$ cd /path/to/test-data
$ cat mysql_error.log
...
single page flush
single page flush
single page flush
...
```

6. Calculate the ratio of three flush types. Refer to the following formulas. You can use `awk` command and `grep` command to easily get the required information.
   - `# of total flush` =  `# of flush list flush` + `# of LRU list flush` + `# of single page flush`
   - `ratio of flush list flush(%)` = `# of flush list flush` / `# of total flush` * 100
   - `ratio of LRU list flush(%)` = `# of LRU list flush` / `# of total flush` * 100
   - `ratio of single page flush(%)` = `# of single page flush` / `# of total flush` * 100

7. After the benchmark ends, shut down the MySQL server:
```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
or
$ sudo killall mysqld
```

8. Vary buffer pool size and repeat the procress, starting from step 3.

## Reference
- https://www.percona.com/blog/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/
- https://github.com/Percona-Lab/tpcc-mysql
