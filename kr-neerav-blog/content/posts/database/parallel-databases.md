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
* The two forms of parallelism are complimentary and can be used simultaneously. Since the number of operations in a query are relatively small compared to number of tuples, intraoperation parallelism scales better.

### Intraoperation Parallelism
#### Parallel Sort
* If the table is range partitioned on the sort key then sort each partition and concatenate it. If table is partitioned on a different way then either range partition and follow the above or a parallel version of external sort merge algorithm.
* In range partition sort the table is partitioned on sort keys and send to different processors. The number of processors involved in the sort can be different than the number of processors/disks storing the initial data. The process of redistribution requires disk I/O and communication. Once the redistribution is complete each processor locally sorts the partition and later on cancatenates the data. The range partition must be done with a good range partition vector to avoid skew.
* In parallel external sort merge each processor locally sorts the partition first. Then it creates local range partition of the sorted partition to be sent to a preidentified processor. The target processor merges the incoming data thus creating a range partitioned table that is sorted. To parallelize the sending of data each processor sends 1 block of each sorted partition to the destination processor. This ensures all partitions are being distributed in parallel.
#### Parallel Join
* Parallel join algorithm attempts to perform join locally on each partition and then the system merges data from each processor to get the final output.
* It works for equi join and the data is partitioned on the join keys used in both the relation. The system can join harsh or range partitioning strategy. Once the relations are partitioned, any join technique can be used (hash join, nested loops, merge join). Thus parallel join can parallize any join technique.
* If one or both of the relation are already partitioned by the join key then it reduces the work related to partitioning greatly.
* Partitioning is done as each processor reads tuples from its local disk, calculates the partition and sends it to the destination processor.
* When using range partitioning system should consider the size of both relations to choose a partition vector. This is to avoid skew for both relations. If only 1 relation is used to identify partition vector then it might result is skew from other relation. For hash partitioning skew is generally unlikely unless the same key is present in a lot of tuples.
* The partition startegy does not work in case of unequal joins as a record in 1 relation can possible join with all records in second relation.
* For unequal join both relation are partitioned based on any strategy/keys. then in relation partition is shared with all partitions of other relation. This repeats for all partitions. Thus all tuples are joined to all tuples in other relation. A special case for this is broadcast join where 1 table is small enough to be replicated to all partitions of other table.
* Partitioned hash join is implemented by partitioning both r and s relation on the same hash function h1 to send the same partition data to the same processor. Then the smaller of the relation is used to create the map/dictionary that is probed for each record in other partition (build). This is called build and probe.
* In parallel nested loop join probe data structure is not created (or an index is created) and each record in build side loops through the partition for a match.
#### Other Relational Operators
* Selection(Filter): for equality condition on partition column only 1 processor is used. For inquality condition it spans multiple partition (incase of range partition) and all partitions if not range partition or not on partition column.
* Duplicate Elimination: it is achieved by sorting done in parallel.
* Projection: already done in parallel
* Aggregation: can parallelize the operation by partitioning on grouping columns, then computing aggregates locally at each processor. Hash or range partioning can be used. We can reduce the cost of transferring records by first locally computing the aggregate value. This is the combine phase in Map Reduce/Spark.
#### Cost of parallel evaluation of operations
* The cost of a parallel operation is not 1/n. Its more than that due to other costs.
* Costs include starting up parallel operation on  each processor, accounting for skew in work distributed to each processor, contention for resources e.g. processor, memory, network and cost of assembling the result.
* The time taken to process the query is decided by the slowest partition and hence skew impacts performance.
* To address skew we could use a balanced range partitioning approach. If a single value has skew then nothing can be done.
### Interoperation Parallelism
* in sequential systems pipelining means not staging the output of 1 operation and instead feeding it as an input to other operation. In parallel databases this is achieved by splitting operations across processors and streaming output of 1 operation from processor A to input of 2nd operation on processor B. This is useful for multi table joins where output of join between 2 tables is streamed to be joined with the 3rd table.
* when degree of parallelism is high partitioning is better than pipelining.
* independent parallelism is when operations are performed in parallel independent of each other.
### Query Optimization
* Query optimizers are more complex for parallel system as they need to take into account additional costs (partitioning, assembling, resource contention)
* Another factor to evaluate is how to parallelize the query i.e. how to parallelize the operation tree and how many processor to assign to it. What operational to execute as sequantial vs pipeline parallel vs independent parallel.
* These decisions fall under scheduled the execution tree.
* long pipelines are bad candidates for resource utilization. the final operation may wait for a long time to get inputs, thus holding up resources and under utilizing them.
* the number of possible plans for parallel exeuction are more and hence search space is high. Heuristics are used. One approach is to first identify an efficient sequantial evaluation plan and then parallelize it.
### Design of parallel system
* storage and processing are main factors as parallel systems are primarily used for storing large volumnes of data and processing for decision support system.
* parallel loading of data from external sources is also an important requirement.
* it must address issues like resilience to failure of some processors or disks and online reorganization of data and schema changes.
* failure of disk or processor is common in parallel system. these are design to continue to execute job if some of the disks/processor fails. disk failure is addressed by reading data from replicas. hence partitions must be replicated to support it. processor failures is accounted by keeping tracking of failed processors and redistributing work to other processors.
* some operations on large table can take a long time e.g. creating an index or adding a column. long downtime is not acceptable. the database supports implementing such operation in online mode.
* For example an index creation process cannot lock the entire table while index is being created. instead it keeps track of changes in the table during the index creation process and applies the changes afterwords.
### Parallelism on multi core processors
* faster processors are power inefficient. this is programmatic in terms of energy cost or battery life in case of mobile. thus mutliple cores are added to a processor instead of making it faster.
* each core is capable of processing data in parallel. 
* if a thread under execution encounters a cache miss and requires data from main memory then processor moves on to the next thread to not waste CPU cycles waiting for the data. This is another form of parallelism available.
