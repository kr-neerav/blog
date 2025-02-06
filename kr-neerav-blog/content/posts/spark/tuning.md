+++
title = 'Tuning'
date = 2025-02-05T19:09:00-08:00
draft = true
+++

## Spark tuning strategy
* Define the goal of tuning
    * For a shared cluster improve resource utilization
    * For a dedicated cluster reduce runtime
* Identify metrics (runtime, memory/CPU usage)
* Identify bottleneck
* Reduce impact of bottleneck

## Spark tuning details
* if cluster does not have enough capacity e.g. there are 50k tasks but cluster has a parallelism of 4 then need to increase capacity.
* reduce amount of data scan
    * partition prunning (skip partitions not needed if input is partitioned)
    * projection pushdown (if input stored in columnar format)
    * predicate pushdown (formats like parquet store additional metadata that can help skip blocks)
    * are input files compressed preferable with a compression algorithm that is splittable
* parallelize tasks
    * parallelize data load from S3 i.e. do we have the right number of partitions defined to read input data.
        * formula
        * of object size < 128MB then number of partitions = number of objects
        * if object size > 128MB then number of partitions = total object size / 128 MB
        * if file unsplitable then number of partitions = number of objects
    * after reading data you  might want to repartition to ensure even distribution as input file sizes might be skewed. too many partitions create too much overhead on the driver. after reading data you can reduce the number of partitions as well.
    * when using adaptive query execution spark will automatically adjust the number of partitions
* optimize shuffles
    * shuffles generate the following costs
        * disk I/O: generates large number of intermediate files on disk
        * network I/O: needs many network connections num of mappers x number of reducers. data is also moved on the network.
        * CPU and memory: sorting and merging data is CPU and memory intensive task.
    * refer to shuffle read/write metrics on executor to understand the amount of shuffle taking place.
    * this operation is the common bottleneck for performance or OOM or Disk full errros.
    * options
        * use dataframe/SQL so catalyst optimizer can optimize the job
        * use broadcast join when applicable. using adaptive query execution automatically takes care of applying these where relevant.
        * side note: shuffle hash join is used when the smaller table partition can fit in memory. shuffle sort merge join is use for joining large datasets.
        * identify data skew from task metrics. use AQE to resolve it.
        * use cache for data that is frequently used.
* minimize overhead on drive
    * too many partition results in additional overhead at drive to plan and track those tasks.
    * too many small files in input results in driver creating too many tasks. Also list operation takes time, setting up connecting with S3 takes time.
* UDF
    * whenever possible use Java/Scala based UDFs so data does not need to be serialized in picked and moved to PythonVM.

