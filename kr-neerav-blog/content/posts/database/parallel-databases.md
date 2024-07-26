+++
title = 'Parallel Databases'
date = 2024-07-26T04:16:16-07:00
draft = true
+++

## Notes
### Introduction
* Data volumnes to process for decision support system has increased to terabytes. Single processor systems are not capable of storing and processing such large volumns at the required rates.
* the set nature of database queries lends itself to parallelization
### I/O Parallelism
* This refers to reducing the time required to fetch data from disk by distributing it across multiple disks. The data is horizontally partitioned.
* There are 3 strategies for it i.e. round robin, hash partitioning and range partitioning.
* Since the data is present on multiple disks transfer rates for reading or writing the entire relation are much faster. However there are other types of scan operations as well like point queries (looking for a specific tuple) and range queries (looking for a range of values)
* Round Robin is suited for queries that read the entire relation.
* Hash partitioning is suited for point queries based on partition attribute. This requires querying only a single disk which is faster as it saves the startup cost of initiating query on multiple disks and other disks are free to process other queries.
* Hash partitioning is also suited for fullt able scan as it results in data being evenly distributed across all disks, unless there is skew on partitioning column. It is not well suited for range based queries as they do not preserve proximity in a range or a point query on non partitioned attribute and require full table scan.
* Range partitioning is well suited for point and range queries on partitioning attributes as the search narrows to only those disks where data is located and other disks are free for other queries. This results in higher throughput while maintaining good response times.
* Range partitioning can result in not using the full compute power of the cluster when the data to be retreived is only presnt on a few disks. This is in contrast of Hash/Round Robin which utilizes the full compute of the cluster. This becomes evident in cases where a large part of the table is retrieved.
* If a table is small ( a few tuples) it should be stored on a single disk. If its large it should be partitioned across multiple disks.
* When a relation is partitioned by a technique other than Round Robin there may be a skew. THere are 2 types of skew attribute value skew and partitioning skew.
* Attribute value skew is when the value in teh partitioning attributes results in uneven distribution of data across partitions. This reduces the speedup of accessing data from disk as it is defined by the largest partition.
* Range partitioned can be balanced by first sorting on partition attribute and then deciding range of partitions. However this incurs initial sorting cost. Additionally if an attribute value has high skew it cannot be addressed by this.
* Range partitioning can also be balanced by using histogram. They are easy to calculate on a sample instead of whole table and small to store so can be calculated on multiple attributes.
### Interquery Parallelism
* Different queries execute on cluster in parallel but the performance of a query is no different than if it was run along. This increases throughput of the cluster.
* Easiest to achive in shared memory architecture. It is complicated in shared disk or shared nothing architecture. There is coordination required among processors for locking and logging. They need to ensure 2 processors don't update a record at same time. when a processor is accessing the data it is the latest. The is called cache coherency and there are algorithms designed to address this.
### Intraquery parallelism
* executing a single query in parallel on multiple processors. this is important to speed up long queries.
* There are 2 ways to do this. Intraoperation parallelism which means performing each operation in parallel e.g a sort operation can be done in parallel on each partition and then the partitions combined to create the sorted table. Interoperation parallelism which means processing different operations in a query execution tree in parallel.
* The two forms of parallelism are complimentary and can be used simultaneously.
* 
