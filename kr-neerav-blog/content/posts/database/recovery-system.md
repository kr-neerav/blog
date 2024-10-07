+++
title = 'Recovery System'
date = 2024-08-24T10:02:13-07:00
draft = false
+++
## Failure Classification
* Transaction Failure: errors that cause at transaction to fail 1) logical error like bad input, data not found etc 2) system error like deadlock.
* System Crash: hard ware malfunction or bug in database software. The content the non volatile system remains intacta and is not corrupted.
* Disk failure: disk block loses its content due to head crash or failure in data transfer.
* recover system consits of 2 parts 1) storing enough information in the system to recover from failure 2) actions taken after failure to recover the database.
## Storage
### Stable Storage Implementation
* replicate the data in several non volatile storage with independent failure mode
* to update the information in a controlled manner such that failure during  data transfer does not damage the needed information.
* Mirrored disks/blocks is a way to replicate data.
* systems using store a copy of blocks in a remove storage updated over computer network. the block transfer either completes or has partition failures (some blocks were successful) or total failure (the failure occurs early and destination block remains intact)
* if a data transfer failure occures the system should recover from it. In mirrored blocks the process of data transfer works as follows. 1) the information is written in primary block 2) when it completes successfully write the information to the secondary block 3) after both blocks are updated mark the data transfer as successful.
* when recovery process is initiated it does the follows 1) it checks the data in each block is same and has no error, then it does nothing 2) if one of the blocks has error it replaces its content with the other block 3) if both blocks have no detectable error but differ in value then the primary block is replaced by back (as this situation is when end user trnsaction did not complete)
* comparing every pair of block is expense. We can reduce the cost by keeping track of block writes that are in progress.
### Data Access
* data stored on disks and is transferred to main memory in the form of blocks.
* conceptually each transaction has a primate work areas in which copies of data items accessed and updated by the transaction are kept.
* when a transactions requests a data item the block for the data item is first searched in buffer and if not present is read from disk to buffer.
* when a transactions writes a data item the block for the data item is first searched in the buffer and if not present it is read from disk to buffer.
* a transation that issues write command might not be immediately written to the disk. it might decide to batch writes or other transctions might still be accessing the block in buffer. This can lead to loss of data in case of crashes. Database takes additional steps to avoid that.
## Recovery and Atomicity
* before updating the database the ouput should first be persisted in a stable storage so we can revoer from system crashes.
* it is persisted to a log in stage storage with format <T, X, V-old, V-new>
* T: transaction identifier
* X: unique identifier of the data item usually a block identifier + offset in block
* V-old, V-new: old and new value of data item
* This entry allows us to redo/undo change as we store the V-old value.
* along with this there are other type of log records to indiciate start of transaction, commit of transaction, abort of transaction.
### Database Modification
* deferred modification if the transaction does not modify the database untill it has committed. immediate modification if the transaction modifies the database while its active.
* deferred modification has overhead that transactions need to make local copies of all updated data items.
### Concurrency control and recovery
* when a data item is updated by transaction recovery algorithm ensures that no other transaction can modify that data item untill the first commits or aborts.
* this is done by acquiring exclusive lock and holding the lock untill transaction commits.
### Redo and Undo Transactions
* in recovery the transactions are executed in the order they are present in log. This ensures that the order in which changes are applied in recovery is the same in which transactions were committed. An incorrect order can result in incorrect value.
* for undo operation the database updates the old values and also creates an entry in log for it. Additionally it adds an abort record in the log to mark the transaction as rolled back.
### Checkpoints
* The recovery algorithm can use the log to retore the database however it will have to parse the entire log. To make it more peformant checkpoints are done which are actions that write all modified blocks to database. During this operations no other transactions process.
* in this process all updates to stable storage and database are done while pausing all other operations. A checkpoint record is also inserted in the log signifying that updates till that point are in database.
## Buffer Management
### Log record Buffering
* writing to log is also batched and done in blocks for performance. A log record is much smaller than a block and multiple log records are batched together before outputting to stable storage.
* also the cost of outputting a block to stable storaege is high given the storage is duplicated.
* requirements for writing to log as follows
* before a transaction can enter commit state the commit record must be entered into stable storage
* before commit record can be entered in stable storage all log records related to transition must be entered in stable storage
* before a block of memroy can be written to db all log records are written to stable storage. This is also called write ahead logging rule. With this rule blocks can be output to disk even when transaction is active as any failures can be addressed by the recovery algorithm using logs. One of the instance this happens when database buffer is full and a block needs to be fetched from disk into buffer.
* when the system finds it necessary to  write to stable storage it combines all log records in a block of storage and writes it. If log records do not fill up the block it writes a partial block. since log is immutable this partial block cannot be filled.
### Database Buffering
* exclusive locks on the blocks are used to ensure no other transactions update the block in buffer while its being written to disk. The lock is immediately released after block is written to disk.
* locks on buffer blocks are independent of blocks used for concurrency control. these locks are for small duration and called latches.
* latches are also used during checkpoint as blocks are written to disk/stable stoage.
* 
