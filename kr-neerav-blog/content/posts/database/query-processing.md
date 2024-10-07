+++
title = 'Query Processing'
date = 2024-08-17T13:52:14-07:00
draft = false
+++
## Overview
* There are 3 steps involved in query processing 1) parsing and translation 2) Optimization 3) Evaluation
* The system parses the SQL to a parse tree that is understandable by database system. in this process it checks the syntax and verifies that the relation mentioned are present in the database.
* The parse tree is then converted into relational algebra expression.
* There are multiple ways to convert SQL to relational algebra and multiple ways to execute the relational aglebra. This is what is done by optimizer.
* The relational algebra can be executed by multiple algorithms that optimizer evaluates.
* The relational algebra can be annoted by hints like algorithm to use or indexes available.
* One the execution plan is finalized it is evaluated/executed.
## Measures of query cost
* cost of individual operations are estimated and combined to calculate the cost of plan.
* it can be measured by disk access, CPU cost or in distributed system as cost of communication.
* ususally disk access is most important in database as its the slowest.
* CPU time is harder to estimate due to time sharing.
* for disk access cost we use number of blocks transferred and number of disk seeks.
* blocks reads and block writes are considered differently as block writes is twice as expensive as block read. This is because disk systems read the block after its written to ensure its written correctly.
* we ignore the effect of block being present in buffer when calculating query cost.
## Selection Operation
* file scan is the lowest level operator to access data.
* linear scan: system scans each file block and tests all records to see if they satisfy the selction. it requires 1 seek if blocks are contiguous. It can be applied on any type of file or the nature of selection.
* primary index, equality on key: use index to retreive a single record
* secondary index, equality: fetching multiple records, worst case each might be in a different block which require its own seek.
* primary index, comparison: the lower bound is searched on index and then file scan till the upper bound is reached (primary index means clustered index). if the conditionis on col <= val then index is not used, a sequential scan of the file is done as there is no lower bound to search on index and you cannot search upper bound and go reverse from there.
* secondary index, comparison: for lower and upper limit the lookup is done on lower value and scan of leaf nodes done sequentially till upper limit is reached. When lower bound is  missing the non leaf index nodes are scanned from the beginning till upper bound is reached. If upper bound is missing then lookup lower bound and search till the end of the leaf node list. additional block access required when records are fetched from the relation. If large number of records are selected then sequential scan of relation might be better than using index.
* conjunctive selection using one index: select one of the conditions that leverages an index with lowest cost of selection. Then in memory check the records fetch to see which ones satisfy other conditions.
* conjunctive selection using composite index: if the filter conditions map to a composite index use that.
* disjunctive selection by union of identifier: for each filter use index to identify potential record pointers and then union them all to get final result set. If even one of the filter does not correspond to a filter then the entire relation needs to be scanned.
## Sorting
* sorting is important because 1) the SQL might have an order by clause 2) many of the operations like join can be implemented efficiently if the input is sorted.
* sorting for in memory relations can be done using quicksort. for relations that don't fit in memory external sort merge algorithm is used.
* the file is consumed in sets of k blocks that fit in buffer. each of them is sorted and written to disk. next we merge the intermediate files to create the final file using N way merge.
* if the number of intermediate files is more than the number of blocks available in buffer then they are merged in parts before final file is generated.
## Join Operation
### Nested Loop Join
* it is a pair of nested for loops between outer relation and inner relation.
* It can be used for any type of join condition and does not require any indices.
* it is expensive since it examples every pair of tuples in the relations.
* the inner relations requires 1 seek per outer loop run. the outer loop requires multiple seeks.
* if one relation fits entirely in memory it is useful to use it as inner relation as then inner relation will be read from disk only once.
* an optimization of this is block based nested loop where each outer block is joined with each inner block instead of records. This reduces number of disk access for both outer and inner relation. each block in the inner relation is read only once for each block in the outer relation, when compared to each tuple in outer relation earlier.
* when neither relations fit in memory use the smaller relation as outer. this reduces the number of seeks required.
* optimizations
* if the inner relation is joined on primary key (i.e. clustered index) then after first match no need to process inner relation further.
* storing more than 1 block of outer relation in buffer. this reduces number of times blocks for inner relation are requested from disk.
* if an index is available on the inner relation we can replace file scan with more efficient index scan. a temporary index can also be created to improve efficiency of join.
### Merge Join
* if the input files are sorted they are join takes place using merging of 2 files.
* each block or tuples is read only once in this algorithm, hence its efficient.
* if any of the input relation is not sorted the cost of sorting can be added to the overall cost to understand if this is an efficient execution plan.
### Hash Join
* the core idea in hash join is tuples in a bucket don't need to be compared with tupes in other buckets for join.
* within each bucket one of the table is converted to a hash index and the other one is looped through with each record probing the hash index. the hash index is built in memory.the hash function used to build this hash index is different from the hash function used to partition data.
* the number of partitions should be high enough such that the build table fits in memory. it is not important for the probe relation to fit in memory. Hence the smaller of the 2 table is chosen as build table.
* if the number of partitions is high and there are no enough blocks in buffer to generate the partitions in 1 go then it is done recursively. First a hash function partitions the relation into the number of blocks available in buffer. Then each bucket is partitioned again with a different hash function. This recursive partition continues till the build table can fit in memory.
* if build table in a bucket is greater than available memory then build table cannot be built in memory. Other blocks need to be chained to it. This generally happens when there are many records with the same value of join key. In this case for that partition other join strategies can be used e.g. block nested loop join.
## Other Operations
* Duplicate Elimination: To remove duplicates relation can be sorted and as part of sort process the duplicates are removed. In external sort algorithm duplicates are eliminated both during sorting the partition and merging the partition. It can also use hash algorithm. When building hash index in each partition value is not inserted in index if already present.
* Projection: it is done by performing projection on each tuple.
* Set Operations: it can be performed by sorting or by hash. These are union, intersection and minus.
* When using sorting relations are first sorted and then combined. In union the are merged but duplicats are removed. In intersection only common between them is output. In minus if a tuple from first relation is present in second it is not output, else it is output.
* When using hash algorithms, the hash index is created. In union entries are not added to hash index if it already exists. In intersection if a tuple in probe is present in hash index then it is output. In minus if a tuple is probe is not present in hash index it is output.
* Outer Join: It can be computed in multiple ways
* first compute inner join using any algorithm and then compute minus operation from outer relation with join table. Then add nulls to right relation and union these.
* For full outer join repeat this for both sides.
* extend nested loops algorithm to output records from outer table that do not match the inner table. However this cannot be used to calculate full outer join.
* merge joins can be extended to compute left/right/full outer join by modifying what to do with records that do not match. since tuples are sorted it is easy to match if a tuple matches/not matches in the other relation. cost estimate for using merge algorithm for outer join is same as using merge algorithm for inner join except cost of writing extra blocks.
* hash joins can be extended to compute outer joins. for left/right outer join the non outer relation is used to created build hash index and outer relation is used to probe it and output record even when there is no match. for full outer join hash index are created for both relations and probe is done from both table to both index.
* Aggregation: can be done using sort or hash algorithm.
* In general hash algorithm can be used when the hash index fits in memroy whereas sort algorithm can be used for any size of relation (since sorting uses external sort merge algorithm that does not have memory contstraint)
## Evaluation of expressions
* expressions are combinations of operators discussed above. they can either be evaluated as one at a time materializing the result at each step or in a pipeline fashion where several operations are evaluated with results of one operations passed to the next.
* cost of materialization plan also includes cost of writing output to disk at each step.
* we can reduce the cost by reducing the number of temporary files produced. we achieve this by combining several relational operations into a pipeline, in which results of 1 operation are passed along to the next operation in the pipeline.
* It has 2 benefits. One to reduce the cost of reading/writing temporary relations. The other is to start generating results quicker.
* pipelines can be exeucted in 2 ways
* demand driven pipeline: the system makes repeated requests for tuples from the operation at the top of the pipeline.
* each operator maintains an iterator with open, next, close functions and also maintains the state of the execution i.e. to know what all it has executed. 
* producer driven pipeline: operator do not wait for requests to produce tuples but instead generate it eagerly. Each operation in the pipeline is a thread the system streams the input and generates stream of output.
* each operator maintains a buffer to store its output as its being streamed. The final operator has an output buffer. If that buffer is full the input stream is paused and wait for output buffer to have some space.
* in a normal database demand driven pipeline is commonly used while in parallel databases producer driven pipeline is commonly used.
* some operators in the pipeline can be blocking operators e.g. the sort merge algorithm cannot begin till both inputs are sorted or hash join cannot begin till hash index is built. 
