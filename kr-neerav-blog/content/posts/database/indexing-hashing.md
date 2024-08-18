+++
title = 'Indexing Hashing'
date = 2024-08-11T10:52:11-07:00
draft = true
+++
* In most cases a query only needs a fraction of tuples. In those cases additional data structures are created to directly reach those records instead of scanning the entire table.
## Basic Concepts
* an index for a file works the same way as index for a book. index is much smaller than the book and is in sorted order for easier searching.
* there are 2 types of indeces ordered indices which are sorted based on search key and hash indices based on uniform distribution of values across buckets.
* there are several factors used to evaluate the type of index to use. access type, access time, insertion time, deletion time, space overhead.
* we can also have more than 1 indices for a file.
## Ordered Indices
* stores the values of search key in sorted order and associates the records that contain it.
* a file can have several indices. if the file is sorted by search keys then it is called clustered index. This is the primary key.
* index entry consists of search and pointers to disk block of records.
* Dense Index: the index consists of an entry for each search key and its record. For dense clustered index it stores value for the first record with search key as other records with same search key will be stored contiguously. For dense non clustered index it must store list of pointers to all records with same search key value.
* Sparse Index: index stores only some of the search key values. It is only applicable for clustered index. the search finds the largest key value less than the search key value and then follows pointers to find the record. an example of sparse index is the index in dictioary on top of the page.
* searching on dense index is faster than searching on sparse index. however sparse index has advantages like less space and less maintainence overhead for insertions/deletions.
* a good compromise is to have a sparse index with an entry for each block because main cost of processing a request is to fetch the block from disk and at the same time reap benefits of sparse index.
* multi level indices: when the index fits in memory searching on it if fast. but for large table the index might not fit in memory which  means multiple disk access is required to find the right record. This is true even if binary search is used to search the index as there are log-n disk block access. since the index is sorted it is possible to create another sparse index on top of index. The sparse index is small and can fit in memory. This reduces number of disk access for the search. If the sparse index is too big, can create another sparse index on top of it. multi level are closely related to tree structure.
### Clustered Index Update
* If a record is inserted/deleted in the file the index also goes through an update. If the record is updated such that the search key for index is updated then the index is updated using delete + insert operation.
#### Insert
* For dense index if its a new search key a new record is inserted in the index at appropriate position.
* If its an existing search key and index stores pointer to a list of records with the same search key value then it updates the list and adds the new record pointer.
* For sparse index which stores the pointer to first record in the block. if record is inserted in existing block and is the first record in the block then index pointer is updated to point to it.
* If its not the first record in the block then no update is done.
#### Delete
* For dense index if the record is the only record in the index then it is deleted from index.
* If there is a list of records with same search key then it is removed from the list. if the system stores pointer to the first record with search key value and the deleted record is the first one then index is updated to point to the next one.
* For sparse index if it points to first record in block and that is the deleted record then index is updated to point to next record.
* If it is not the first record in block then no action is performed.
* If index does not contain the search key value then no action is performed.
### Secondary indices
* Secondary index must be dense i.e. index entry for every search key value and a pointer to every record on the file. They are equivalent to dense index which stores pointers to all records.
* They follow the same process for insert/delete as of dense index which stores pointesr to all records.
* each secondary index increases the overhead of inserts/deletes/updates on the file and hence must be chosen wisely.
### Composite indices
* The same approach can be applied to search key that contains multiple attributes. They are stored as a list/tuple in the index.
* The order of attributes is important as it defines the order of search key.
## B+ Tree Index Files
* The disadvantage of index sequential file (e.g. the one storing the index) is that performance degrades as file grows both for index lookups and sequntial scans. The degradation can be remedied through reorganization of file but frequent reorganization is costly.
* B+ Tree structure is frequently used for most index structures which maintains efficiency despite inserts/deletes. Each node has n/2 to n branches and is balanced tree that ensures path from root to leaf is of same length for all nodes.
* There is overhead in insert/updates but it is acceptable as reorganizations are not required. there is additional space overhead as nodes might be half empty but it is acceptable given the performance of the index.
### Structure of B+ Tree
* its equivalent to a multi level index but has a different structure.
* Each node contains n-1 keys and n pointers.
* For leaf node key is the search key value and pointer is the pointer to record in table. The last pointer is pointer to next leaf node. They way keys/pointeres are arranged are in pairs as pointer to values < key and key. At the end a pointer is stored that points to keys > last key value in node.
* If there are duplicates of search key value in table then leaf node contains multiple entries for it.
* B+ Tree are always balanced
### Queries on B+ Tree
* To query the tree each node is traversed till we reach a leaf node. In each node the key is searched which has max value <= search key. Then the pointer to the next key is used to traverse the tree. If search key is greater than all keys in the node then last pointer is followed. If search key is less than first key in node then the left most pointer is followed.
* At the leaf node if a search key value is found then pointer assigned to it is followed. If not found then search key does not exist.
* If there are more than 1 records with same search key then the node is traversed till a key > search key is discovered. If the last key of the node is also same as search key then next left node is traversed.
* The index can also be used for range queries (L,U). First index searches for L and then traverses records in leaf node untill value U.
* the search complexity is O(log-n^N) where n is the number of records in table and n is the number of keys in a node.
* B+ tree are fatter and shorter than binary tree as each node has multiple children. This supports faster querying as height of tree is short.
### Updates on B+ Tree
* Updates are more complex than lookup as it needs to account for split/coalesce of ndoes dependending on the size of the node.
* With insert we find the leaf node where search key should appear and insert the key with pointer to record.
* If the leaf node is full then we split it up into 2 leaf nodes and divide the keys equally between them. Then a new search key and pointer is inserted into the parent. 
* if the parent is also full then it is split like the leaf node and keys divided between the 2. This happens recursively. The split for parent is different than split for child as the search key values that lie at the split are moved to the right node.
* With delete we find the leaf node containing the entry to be deleted and delete the entry from it. all entries to the right of the search key in the leaf node needs to be shifted left by 1 to ensure no gaps in node. If there are multiple records related to the search key then all are scanned to identify the record to be deleted.
* If the deletion of search key results in leaf node size to decrease beyond n/2 then first it tries to merge this with either left or right sibling. If that is not possible then it redistributes the keys between the 2 siblings. This process recursively takes place for parents if their size falls below n/2.
### Non unique search keys
* if an index is built on non unique search keys deletion is inefficient as it needs to scan through all records with same search key.
* To make it efficient a unique identifier is added to a search key, thus the search operation now results in a single search key. However this is space efficient.
* An idea was, instead of storing each search key + unique identifier create a bucket where we store pointer to all records with same search key. This is space efficient but complicaes the code for search/insertion/deletion. Additionally if this bucket is stored in a separate page then an additional page lookup is required for accessing data. Also this does not address the problem of scanning the bucket list to identify the records to be deleted.
### Complexity of B+ Tree
* In real life seraching B+ tree is faster. As the non leaf loads are less (given high fanout factor) and accessed more frequently, they are likely to be in database buffer already. Thus access to record only requires 1 disk operation which is recovering the leaf node.
### B+ Tree file organization
* a large file is stored as a B+Tree with leaf nodes containing the actual data instead of pointers to records. This is helpful for large objects like clob/blob. When we update such a large object, instead of writing the whole file only parts of B+Tree are rewritten. Additionally this type of file starts of as being stored in contiguous blocks but with updstes its stored at different places on the disk. Occasionally it needs to be defragmented.
### Indexing String
* Strings can be long and of variable length. hence creating an index on them, the number of keys in a node can vary dependending upon the length of key. Also the fanout can be low if the key length is large. The way to workaround this is to store prefix of string in non leaf nodes, thus saving space.
### Bulk loading B+ Tree
* inserting into existing B+ Tree one at a time is efficient if there are a few inserts only. But doing it in bulk can be time consuming as each index entry requires access to disk
* first create a temporary file containing index entries for lead node only. Now sort this file.
* Using sorted file as input start generating index. Since once a leaf node is written it will not be accessed again the number of disk access is reduce.
* Another approach is to use the sorted file to generate the index bottom up. The sorted file is broken up into blocks. The minimum entry from each block and pointer to it is written to another file that represents one level higher in the tree. This process is repeated till we reach the root node.
### B-Tree
* B tree is a special version of B+ Tree in which search keys are not repeated in the tree. So each search key now stores 2 pointers 1  to children and 1 to the record. Thus if a seaarch key if found early it does not need to traverse the tree.
* however this is space inefficient and now tree depth is more.
* Usually there are n times as many records in leaf node compared to non leaf node. Hence the benefit of ending search early is not significant and the drawback of less fanout has a bigger impact.
### Flash memory
* B+ Tree were originally designed for magnetic disks. With flash memory the look speeds is significantly higher.
* however the downside is updates are slow as flash memory needs rewrite of entire page (which is very large) compared to a single block.
### Multiple key access
* If there are multiple indices then we can query more than 1 index and find intersection of pointers from those to identify records that need to be read.
* This scenario is not helpful if individual lookups resulted in large number of pointers but the intersection is small. We end up reading a lot of blocks.
* This limitation is addressed by bitmap indeces (discussed later)
* Another approach is to create a composite index. It requires new type of data structures like BitMap or RTree.
* A common approach used is to store some additional frequently used value with the pointer in the leaf node. This ensures that querying that attribute along with index does not require to read the relation.
## Static Hashing
* hash based indices avoid the need for an index data structure, but do not store the index in any order.
* the hash value of the search key is mapped to a predefined number of buckets. Each butket is a block but can be smaller or larger. To search for the record the entire bucket must be scanned.
* More than 1 key can land in a bucket and in those case more records need to be scanned in hte bucket for searching.
* Just like B+ Tree file organization we have hash file organization.
* a good hash functions distributes the records uniformly across buckets.
* skew can occur for 2 reasons 1) multiple records may have same key 2) multiple keys map to the same bucket.
* if a bucket is full and records need to be inserted into it then another bucket is chained to it and new records stored there. As many buckets can be chained together as required.
* the hash function and number of buckets are chosen at the beginning and cannot be changed.
* hash index is used for secondary indices as primary index is clustered and there is no benefit of created a hash index of it.
## Dynamic Hashing
* The challenge with static hashing is the as database grows perforance degrades because hash function and number of buckets were not designed for that scale.
* One can recreate the file but it takes signficant resources to do that.
* estendable hashing is a technique that copes up with changes in database  size by splitting/coalescing buckets as database grows/shrinks. Since only 1 bucket is modified the performance overhead is relatively low.
* a hash function generates a large number of buckets e.g. 32 bits however only a small prefix of this is used to map to the prefix. when a bucket grows in size it is split and another bit is included in prefix for search. This doubles the bucket address space each time. however multiple search key prefix can point to a single bucket. This ensures space efficiency.
* if all the keys in the bucket have the same value, no amount of splitting will help. in this case overflow bucket is used.
* when a bucket is split hash function needs to be recalculated for only records in that bucket.
* the disadvantage of extendable hashing is that there is an additional hop to reach the bucket compared to static hashing where you directly reach the bucket.
* hash based indices are preferred as the lookup takes constant time compared to log in orderd indices. The only case when ordered indcies are prefered in when range queries are frequent. Additionally hash based structure if prefered for temporary files generated during query processing (hash join).
## Bitmap Indices
* specialized index designed for querying using multiple low cardinality keys.
* for this index the record must be numbered 1 to n and given a record i it must be easy to locate it. This is achieveable if the records are of fixed size and allocated on consecutive blocks of a file.The record number can then be easily translated into block number.
* each bitmap is a sequence of bits with 1 representing presence of a value and 0 representing absence (similar to one hot encoding)
* querying a single bitmap is not helpful, its beneficial when multiple are queried together to reduce the number of records retrieved.
* bitmaps are very small compared to the table.
* bitmaps can also be used for count queries and don't require accessing records for it.
* bitmaps are combine using bitwise and instructions and thus is very fast.
* deletion of records creates gap in sequence of records and shifting records will be very costly. a deletion bit map is used to identify which records are deleted.
