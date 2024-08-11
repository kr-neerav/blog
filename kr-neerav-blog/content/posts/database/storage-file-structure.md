+++
title = 'Storage File Structure'
date = 2024-08-10T09:32:37-07:00
draft = true
+++

## Overview of Physical Storage Media
* Cache: fastest and most costly and relatively small
* Main Memory: data that is available to be operated on and is volatile
* Flash Memory: non volatile. is used in SSDs as a replacement for magnetic disks
* Magnetic Disk Storage: primary medium for long term online storage. entire database is stored in it. system must move data to main memory for operation.
* Optical Storage: Compact Disk (CD)
* Tape Storage: used for backup and archival, cheaper than disks but access is much slower as tape must be accessed sequentially from beginning. This is sequential access compared to disk storage that is direct access. Tapes have high capacity.
* The fastest storage media cache and memory are known as primary storage. The next level of magnetic tapes are known as secondary storage or online storage. The next level is known as offline storage. 
## Magnetic Disk and Flash Storage
* capacity of storage disks are increasing but requirements of applications are growing faster than disk capacity and they need to use multiple disks.
### Physical characteristics of Disks
* Disk -> Platter -> Tracks -> Sectors
* A sector is the smallest unit of information that can be read/written to the disk.Its about 512 bytes.
* Heads on all platers move together. If head is on i-th track of a platter, all heads are on i-th tracks of the platters and they form the i-th cylinder
* disk controllers interfaces between computer and disks. they read/write sectors. They also attach checksum to a sector as its written, when the data is read back the controller calculates the checksum to ensure data has not been corrupted. If there is a checksum mismatch, the controller tries to read multiple times after which it reports a read failure.
* they also remap bad sectors to ensure no read/write operations take place there. this mapping is stored on disk.
### Performance Measure of Disks
* Access time: time from when a read/write request is issues to when the data transfer begins. Time the disk head takes to move to the right track and wait for right sector.
* Average seek time: avg of seek times measured of random requests.
* Data Transfer Rate: rate at which data is transferred to/from disk.
* Mean Time to Failure: measure of reliability of a disk
### Optimization of Disk Block Access
* Block is a logical unit consisting of fixed number of contiguous sectors. Each block has an unique address called block number. It is also referred to as page.
* Sequential access is when contiguous blocks/pages are requested. The seek time is amortized across number of blocks read. Random access is when each block has an associated seek time as the blocks are not sequential. Transfer rate is significantly lower in random access when compared to seuqntial access.
* Buffering: blocks tha are read from disk are stored temporarily in a memory buffer to satisfy future requests. It is done at both OS level and database level.
* Read-Ahead: When a block is requested, consecutive blocks are read and stored in buffer incase there is a request to acess then, it will be faster.
* Scheduling: if multiple blocks need to be accessed it makes sense to save them on a disk cylinder to avoid seek times for all but one. Also disk controllers use scheduling algorithm like elevator algorithm to minimize the seek time by scheduling requests as an elevetor i.e order them such that those can be covered as disk arm is moving in 1 direction.
* File organization: depending on type of read operations file should be stored on disk. If a file requires sequential access then its blocks should be consecutive to each other. if not consecutive then large number of blocks should be contiguous to avoid seek time. Over time appends to a file might result in it being spread across many non contiguous blocks. The file is fragmented and needs to be defragmented by creating a new copy and storing it in contiguous blocks.
* Log disk: reduce writing latencies by writing to a sequential log, eliminating seek time
* Flash Storage: faster random access as no mechnical instruments involved in seek time. Throughput is less compared to magnetic disk. SSD combine multiple flash storage to improve throughput. Write operations are slow as an entire page needs to be written at a new place and previous page marked to be erased. 
## RAID
* having large number of disks presents opportunities for improving rate at which data can be read or written, if disks are operated in parallel. it also improves reliability by redundantly storing information.
* a disk organization technique to improve reliability and performance is redundant array of independent disks (RAID)
### Improvement of Reliability via Redundancy
* repilcating data in 2 disks exponentially increases the redundancy. mean time to repair a disk is the same the mean time to data loss is increased.
* Mirroring is a technique of replicating entire disk and keeping them both upto date. if writing to both copies and one of the disk fails it might result in failure of transaction. hence write to 1 disk first and then update the other one later.
### Improvement of Performance via Parallelism
* Bit Level Stripping: Different bits of a byte are stored on different disks. Hence transfer rate of data increased.
* Block level stripping: different block stored on different disks. When reading a large file block stripping fetches n blocks at a time. When requesting a single block then other disks are free to perform other tasks.
* It is the most common type of stripping.
### RAID Levels
* Mirroring provides high reliability but its expensive. Stripping provides high data transfer rates bu t does not provide reliability. Different schemes have been devised that combine stripping with parity bits to provide low cost storage.
* RAID level 0: Stripping at level of blocks but no redundancy (normal disk usage)
* RAID level 1: block stripping and disk mirroring (entire disk replica).
* RAID level 2: Parity bit is used in memory to detect 1 bit errors i.e. the partity bit stores if the number of 1s in a byte is even or odd. If one of the bit flips then it will not match parity bit, if the parity bit flips then it will not match. Hence is it efficient for identifying 1 bit errors. For correcting the error 2 or more additional bits need to be saved. So for a array of 4 disks 3 additional disks are required for redundancy compared to 4 in mirroring.
## File Organization
* a database is mapped to multiple files in the system that are managed by the OS. A file is logically organized as a sequence of records and they are mapped to blocks on disk.
* Each file is partitioned into fixed length storage units called blocks of size 4KB or 8KB.
* A block may contain several records. Assume a record is less than the size of a block.
### Fiexed length record
* Each record is of fixed size. Simpler to implement than variable length record.
* If a record can only fit partially on a block then it is moved to next block and some space is wasted in current block.
* If a record is deleted then its added to the free list. The header of the file stores a pointer to the free list. when a record is to be inserted it checks the header to identify an empty space and upadtes header to the next element in the free list.
### Variable length record
* Its for database that allow multiple type of record types to be stored or allow variable lengths for one or more fields or allow repeating fields such as arrays.
* the fixed lengths fields are saved at the beginning of the record followed by variable length fields. Each variable length field has a (offset, length) metdata stored that is used by DB to identified where the variable length field begins and what is its length. The record also stores a null bit map with 1 bit representing each attribute. This is used to identify which attributes have null value. The fixed length attributes that are null are skipped when reading the records. In case of sparse table the physical space for null columns might also be skipped for storage efficiency.
* next how to store variable length records in a block. use slotted page structure.
* In this structure the records are stored contiguously from the right. The header of the block stores an array that points to the beginning of each record (offset). Between header and records is the free space. When a record is deleted the records are moved inside the block so they are contiguous. Thus databsae only knows the block where a record is stored and not the exact location of the record in the block. This allows defragmentation of the block without impacting any pointers at database level.
* Large objects like blob, clob are not stored with the record. Instead they are stored in a special file and pointer to that file is stored with the record. The special file is stored as a B+ Tree that allows reading an entire record or specified byte range or insert/delete part of the object.
## Organization of Records in Files
### Heap file organization
* Any record can be stored anywhere in the file. typically a single file is used for each record.
### Sequential file organization
* records are stored in sequential order according to the value of the search key.
* Phsically also its stored on disk in this order. Each records has a pointer to the next one.  As new records are inserted they are stored in overflow space and pointers updated in the records of the original table. If a record is deleted the pointers are updated. Overtime the physical sort order of the table is no longer helpful and its jumping between table and overflow space. At that time the table needs to be recreated in sort order.
### Multitable clustering file organization
* Usually a different file is used for each relation. This lets DBMS leverage OS file system capabilities. 
* In some cases multitable tables can be stored in a single file. in this case a single file is managed by DBMS and it internally manages storing records in each.
* In some cases related record are stored in a single block. this enables faster join between 2 tables.
* However it reduces performance of single table queries as it has to scan through more blocks now.
## Database Buffer
* dbms wants to reduce the transfer of blocks from disk to memory. One technique it uses is of database buffer i.e. a section of main memory that stores blocks from disk.
* database buffer can point to a new version of the data in disk block when compared to disk blocks. 
### Buffer Manager
* When a program requests a block from disk buffer manager checks if the block is present in the buffer and returns its address.
* if block is not in buffer it replaces an existing block in buffer with the block read from disk and returns its address.
* when it replaces an existing block it writes it to disk only when it has been updated since it was read.
* operating system uses Least Recently Used block to replace. however there are scenarios when a buffer should not be evicted e.g when its updated. in such cases the block is pinned to the buffer.
* other times a write to disk might be forced, even when a block is not requested by program, for situations that the data might be lost due to crash.
### Buffer Replacement Strategy
* OS does not have a good strategy to understand what blocks will be required next and hence it uses LRU as its strategy.
* however database has good understanding of next steps which it can use to decide which blocks to evict.
* In nested loops join an efficient strategy is to use Most Recently Used (MRU) schema. This is because once the block for outer loop has been used it will not be used again. Once a block for inner loop is used then it will not be used untill the inner loop finishes.
* However DB must pin the current block so its not evicted before its processing is complete.
* Other compoents like concurrency control or crash recovery might influence what blocks to keep. e.g. a crash recovery might block updates to a block till backup is in progress.

