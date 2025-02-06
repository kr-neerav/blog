+++
title = 'Mpp_mr_systems'
date = 2025-02-06T03:46:36-08:00
draft = true
+++
## Classic Parallel Database Systems
* Parallel databases were created to solve for large scale data analytics. They were primarily based on relational data model and supported declarative query language SQL.
* They achieved performance and scalability by partitioning table across nodes in shared nothing architecture. This allowed running relational operators like filter, join, aggregation in parallel.
* Challenes with parallel databases 1) hurdles in terms of scalability and total cost of ownership as coupled storage and compute 2) hard to fit semi structured and unstructured data into relational model supported by MPP 3) push to have access to data with reduced SLA i.e. micro batches.
### Categorization of systems
* columnar systems category: storing data in columnar format instead of row format. reduced I/O.
* Map Reduce systems: combining multiple standalong distributed systems and composing them together to run analytic tasks on large datasets e.g HDFS for storage, Map Reduce for processing.
* Dataflow system category: execution techniques of map redeuce systems were suboptimal for some data analysis tasks like relational operations, iterative ML, graph processing. These were addressed by flexible data flow based execution model like Spark.
### Categorization of System Features
* Data Model and Interfaces: logical structure of data and how it is stored, organized and processed by system. MPP supports relational model whereas data flow and map reduce support any arbitary format.
* Storage Layer: MPP systems used integrated and specialized storage layer that is tightly coupled with execution engine wheras map reduce uses an independent storage system.
* Execution Engine: responsible for running the execution plan, parallelizing the computation, handling machine failures and setting up inter machine communication.
* Query Optimization: process to determine the most efficient way to execute a given query. we evaluate 1) space of possible execution plans 2) type of query optimization e.g. cost based vs rule based 3) type of cost modelling technique e.g. analytical models vs ML based 4) maturity of optimization technique e.g. fully automated vs manual
* Scheduling: scheduling decisions like where to run each computation, scheduling inter node data transfers
* Resource Management: efficient and effective use of cluster's resources. also elastic properties that allow cluster to dynamically add/remove resources as needed
* Falut Tolerance: to address machine failures to allow to continue processing, possibly with graceful degradation. examples are restarting failed tasks, recovering data due to machine failure or corruption, using speculative execution to avoid stragglers.
* System Administration: activities to keep the system running smoothly like performing monitoring and tuning, diagnosing cause of poor performance or failures, capacity planning and system recovery.

## Classic Parallel Databaes Systems
* When single system databases could not handle the workload posed by data analytics 3 architectures emerged.
    * shared memory: all processors share memory and disk. this has limited scalability because access to memory becomes a botteneck.
    * shared disk: each processor has its private memory but shared disks. This becomes expensive at scale due to the complexity of connecting all processors to disk. There are also scalability limits.
    * shared nothing: this has been most viable over the years. each processor has its own private memory and disk. only shared resource is the communication network.
### Data Model and Interfaces
* supports relational model which is simple. a well defined schema helps with cost based query optimization and keep data error free incase of data entry errors or bugs.
* however the data model is rigid which results in longer development time due to the need to define schema upgront.
* uses SQL has query language. it is declarative, easy to learn. supports UDFs for specifying analysis tasks that are not easily expressed in SQL.
### Storage Layer
* SQL has the benefit of data independence i.e it can be executed correctly irrespective of how the data in tables in stored.
* system partitions data (breaks it into disjoint fragments) and assigns (distributes) partitions across nodes in the cluster.
* horizontal vs vertical partitions. vertical partitions is more common in columnar databases
* benefits of partitions in parallel systems
    * efficient pruning of unneeded data
    * parallel data access during query processing
    * faster data loading
    * prioritize data storage based on fast/slow access
    * simplifier administrative tasks
* Types of partitioning
    * Range partitioning: data partitioned based on predefined ranges
    * hash partitioning: data paritioned based on result of a hash function
    * Random (Round Robin) partitioning: data partitioned randomly
* partition sizes can be skewed if the keys used in partitioning have a skewed distribution
* 3 factors considered as part of partition assignmnet i.e. which partition is assigned to which node
    * degree of declustering: the number of nodes the table needs to be distributed to. full declustering (maybe by default) means table is distributed across all nodes. Helpful for large tables. partial declustering means distributed across a set of nodes.
    * collocation: having partitions of same keys in the same node for multiple tables. for hash partitioning this means same hash function, same key and same number of buckets. For range partitioning this means same key and same partition ranges.
    * replication: store replicas of table ,which can be partitioned differently to support different queries. replicas can also be of partitions in which case the replica has the same partitioning strategy as the original. replic also helps with reliability
### Execution Engine
* it is responsible for breaking query into multiple tasks and orchestrating the execution of these tasks.
* there is a coordinator task (leader node in Redshift) that invokes query optimizer, scheduling tasks.
* join strategies
    * if table are collocated join locally
    * if table are not collocated and one of hte table is small use broadcast join
    * if both tables are large repartition them so tuples are collocated and then join
    * if there is skew use the design pattern for handling skew used in Hive and Spark.
### Query Optimization
* steps involved
    * plan selection: choosing an efficient execution plan for the query
    * scheduling: generating executable tasks and choosing nodes where task will run on
    * Resource allocation: determining how much CPU, memory, I/O resources to allocate to the tasks
* finding an efficient plan is more complex that a single system database as now the optimizer needs to take into account distribution of data e.g. join ordering might be different if 2 out of 4 tables are collocated or semi join can be used to reduce the network cost.
* semi join in an approach to filter out the large table by filtering on the join keys of hte small table. works when cost to repartition large table is significantly more than cost to broadcast join keys of small table
* to address the increased complexity due to parallelism it is proposed to use adaptive plan selection which is updated as new informaiton is discovered during execution.
### Scheduling
* plan is broken down into tasks which are scheduled to run on nodes.
* the tasks form a DAG based on producer consumer relationships.
* one approach use to decide which task to schedule on which node is to choose the node on which data resides. this minimizes data transfer.
* the scheduling algorithm can be static or dynamic. static alrogithm decides the entire schedule upfront. this has a challenge if one of the nodes is busy it cannot use an alternate node to process the data. dynamic scheduling can accmodate such changes.
### Resource Management
* at cluster level this is done through techniques like workload differentiation (identifying short running and long running queries and allocating resoruces accordingly).
* scaling system up as load increases. when adding new nodes the data needs to be repartitioned. this is avoided in redshift by having more number of partitions initially that can be reassigned to new nodes as needed. thus only assignmnet of partitions changes and not the partitions itself.
### Fault Tolerance
* there are more causes of failures in parallel systems when compared to centralized systems.
* task failure due to hardware issues, software bug, killing of tasks due to failure of other tasks or for using more memory. Redshift retries task for a few times for address any transient errors like network glitches. for data errors it will fail the query. Retry is helpful for long running jobs.

## Columnar Database Systems
### Data Model and Interfaces
* use relational data model containing tuples with fixed schema
* use SQL as query interfact. the main difference from MPP systems is use of different storage format for data.
### Storage layer
* it is common to use large disk pages to amortize disk seeks. for redshift this is 1MB
* each column is stored as a separate file.
* an inefficient approach was to store <k,v> in each file representing the key of the record. this increases storage requirements, reduces compression efficiency.
* another approach is to derive tuple identifier using position of each attribute value in the file
* additional metadata is stored for each block like min/max values of column. this enable execution engine to skip blocks that are not required.
* columnar data layout allows us to store columns densely on disk using lightweight compression. The reason is that data is a column is similar (datatype, values) thus enabling better compression.
* compression techniques
    * Run length encoding (RLE): replace sequence of same values with the value and number of occurence. very good for low cardinality columns that are sorted.
    * Delta Value: each attribute value is stored as a difference from smallest value. useful when differences can be stored with smaller bytes than the original value. the same approach can be applied in a block i.e. all value in the block are stored as deltas from the minimum value in the block. this technique is useful for many valued unsorted integer
    * Compressed Delta Range: similar to delta but instead of difference from smallest value its difference from previous value. usefl for sorted integer, floating point values.
    * Dictionary: distinct values are stored in dictionary which assigns a short code to each.  the actual values are replaced with short codes. there is a block oriented approach which creates a dictionary per block. this is good general purpose scheme most useful for few valued unsorted columns
    * Bitmap: column is represented by a sequnce of tuples <v,b> where v is the value and b is a bitmap indicating positions where value is stored. bitmap is further compressed using RLE. Useful for few valued columns.
* insertion/update is more costly as columns spared across multiple files which need to be updated.
* also due to densely packed columns insertions to tuples within a disk block is nearly impossible. the approach used is to buffer writes temporarily and then apply them in bulk to update the columnar data layout.
* thus there is a read optimized store (columnar) and write optimized store (row oriented)
* deletes are marked on read store, updates = deleted in read store and insert in write store.
* all queries that read data access both read and write store and combine data.
* partitioning technique similar to MPP systems. same is true for declustering, assignment and replication.
### Execution Engine
* data is decompressed only when necessary to avoid the cost of decompression which will also increase the memory required to store this data.
* some relational operators can be implemented on top of compressed data e.g. filters can be validated on dictionary values.
* vectorized processing is faster as opposed to tuple at a time. the instructions are loaded once in memory and an array of column values are processed in 1 go.
* tuples are materialized as late as possible, preferably only when results need to be retruned. this is done as tuple reconstruction is costly operation which requires joining multiple columns.
### Query Optimization
* techniques similar to MPP systems.
* some additional factors like late materialization, choosing operators that work on compressed data.
### Scheduling
* techniques similar to MPP systems
### Resource management
* more challenges than MPP systems. too detailed and can be skipped.
### Fault Tolerance
* similar to MPP systems

## Map Reduce Systems
* the programming model is highly flexible but too low level ffor routine use by practitioners such as data analysts, scienctists.
### Data Model and Interfaces
* primarily process data from files allowing data to be in any format. hence they can process structured, semi structured and unstructured format.
* users can create their own processing logic by specifying a customized map and reduce function.
* supports multiple programming languages like Java, C++, Python
### Storage Layer
* it is an independent distributed file system like HDFS.
* it focuses on batch processing rather than interactive use by users. emphasis is on high throughput rather than low latency.
* HDFS has master/slave architecture with a single NameNode and multiple DataNode. NameNode manages the file system namespace and regulates access by clients. DataNodes store the actual data and fulfill read/write requests.
* internally file is split into 1 or more blocks and replicated for fault tolerance.
* It enables high level systems to introduce their own features. Hive partitions data into directors and stores this metadata in a Metastore along with schema and statistics.
* there is no concept of collocation in HDFS.
* can internally store data in columar format i.e. withing each block/file store the data in columnar format.
### Execution engine
* users define computation on massive data in terms of map and reduce functions and system automatically parallelizes the computation across nodes, handles failures and schedules tasks.
### Query Optimization
* this is a new system and hence its performance can be poorer when compared to database system running same amount of cluster resoruces.
* it relies on manual tuning and lack of cost based optimizer results in missed opportunities for better join or partitioning.
* higher level systems provide optimizer like Hive
* performance is affected by configuration parameters like degree of parallelism, use of compression.
### Scheduling
* goal is to maximize data locality i.e. schedule task where data resides
* it had a FIFO schedulter for jobs. Higher level systems introduced concepts of priority queue and schedulers like Fair Scheduler and Capacity Scheduler
* since tasks are divided across multiple nodes some might be slower. To address this speculative execution is used i.e. scheduling some redundant tasks on nodes that are not currently busy. whichevery copy of the task finishes first becomes the definitive copy and the other one is abandoned.

## Dataflow Systems
* model the end to end data flow
### Data Model and Interfaces
* flexible data models that support structured, semi structured and unstructured data format
* the processing of data is defined as a set of computational vertices that define local processing on a partition of data and a set of communication edges that define data transfers between vertices. These vertices and edges are arranged as a DAG.
* query interface consists of declarative languages like DQL, functional programming interfaces like Spark. 
* operators process partitions of input data and produce partitions of output data. connectors repartition operator outputs to make the newly produced partitions available for cnosuming operators.
### Storage Layer
* distriburted file system like HDFS which are optimized for large sequential reads and writes and serve once written read multiple type of workloads.
* wide columnar store designed on top of distributed file system like HBase, BigTables.
* in memory data storage like RDD. each RDD is backed by external storage system or another RDD.
* several data serialization formats are available for storing and transferring data independent of storage layer used.
### Execution Engine
* execute DAGs
* vertices can have arbitary number of inputs and outputs.
### Query Optimization
* Static Rules based optimization: removing unnecessary columns, pushing down selection predicates, pre aggregating
* Dynamic Runtime optimization: mutating execution graph based on runtime information such as statistics.
* Cost based query optimization: applied after rule based optimization.
### Scheduling
* schedule tasks on node such that high data locality is can be obtained.
* spark uses FIFO scheduler
