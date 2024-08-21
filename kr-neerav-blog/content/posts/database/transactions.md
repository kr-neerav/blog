+++
title = 'Transactions'
date = 2024-08-19T05:23:50-07:00
draft = true
+++
## Transaction Concept
* often a collection of operations on a database appears as a single unit from the point of view of the database user. this collection is called a transaction and either all operations must succeed or all must fail.
* it is delimited by statement begin transaction and end transaction.
* if the transaction begins executing but fails in the middle any changes that transaction has done must be undone. this must be done whether the transaction failed or system crashed. This all or none property is called atomicity.
* each transaction executes without interference from other concurrent transactions as if they executed serially one after the other. This property is called isoltion.
* after a transaction is complete it must persist across crashes. this property is called durability.
* Database is in a consisten state before and after the transaction. This goes beyond the data integreity constraint defined like PK, FK, check constraints etc. One example is sum of balance of accounts remain same of credit/debit is complete. Ensuring this consistency is the responsibility of the application. 
## A simple transaction model
* consider simple operations read(X) which transfers data from database to variable in buffer and write(X) which transfers the values in main memory buffer to the database.
* we assume write operation updates database immediately. in real world write operation may be temporarily stored elsewhere and executed on disk later.
* the responsibility of atomicity is with database system. To ensure this database must store old values to have the ability to restore them in case of failures.
* For durability we assume data in main memory can be lost but not the one stored on disk. Thus updates on the transaction are written to disk before transaction completes and this information is sufficient for database to reconstruct the updates when the database is restarted after a failure. The recovery system is responsible for ensuring durability.
* even if consistency, atomicity, durability are ensured if several transactions are executed concurrently their operations may interleave in some undesirable way. One way to avoid this os to perform concurrent transactions serially but it loses the performance benefits of concurrency. The concurrency control system is responsible for ensuring this property.
## Storage Structure
* Stable storage: information residing in stable storage is not lost. This is ensured by storing data in mulitple non volatile storage (disks) which have independent failure modes. For some applications local disk is considered as stoable storage. For others where transaction are highly important and valuable mulitple copies of data is considered stable storage.
## Transaciton Atomicity and Durability
* a transaction that is failed is rolled back and that completes is committed.
* before the transaction completes it is recorded in the log with details like id of the transaction, id of the record, old value, new value. This provides the ability of recovery system to redo or undo a transaction when it restarts after a crash.
* if a transaction completes it cannot be undone. a compensating transaction is required to bring database back to previous state. This is not done by the database but is the responsibility of the application.
* side effects should be handled differently e.g. writes on user screen only takes place when trasnaction commits or in case of ATM if final dispensing of cash failed it does not make sense to dispense cash upon restart as the user might have left.
## Transaction Isolation
* Concurrency is helpful to increase the throughput of the system i.e. number of transactions it completes in a given time and resource utilization of the system. It also reduces the wait time for users e.g. a long running transaction does not block a short transaction from user.
* example of 2 transactions that can cause challenge when executing concurrently e.g. one transaction adding to account balance and one transaction increasing account balance by a given %.
* The execution sequence of steps/instructions in a transaction is called schedules. They represent the chronological order in which the instructions must be executed.
* When there are multiple transactions CPU does a context switch in which it performs a few instructions from 1 transaction and then switches to perform a few instructions from other transaction and so on.
* Several execution sequence are now possible as the instructions are interleaved. In general it is not possible to predict how many instructions of a transaction will be executed before the CPU switches to another transaction.
* if the control of concurrent exeuction if left to the OS many possible schedules including one that leave database in inconsistent state are possible. It is the responsibility of concurrent control component of database that only those schedules are executed that do not result in the database being in inconsistent state.
* this is done by making sure the schedule that is executed is equivalent to the schedule such that transactions were executed serially. Such schedules are called serializable schedules.
## Serializability
* transaction is broken down into instructions of read(Q) and write(Q). these are the instructions that need to be considered for deciding serializability. Other instructions of doing any operation of Q in memory is not of concern to decide serializability.
* instructions of 2 transactions are swapped such that there will be no impact on the ouput. If the 2 instructions from 2 transactions after swapping becomes serial then the transactions are serializable else not serializable.
* the algorithm for it creates a graph on instructions and edges between certain instructions if one follows the other e.g. T1-write(Q) before T2-read(Q), T1-read(Q) before T2-write(Q), T1-write(Q) before T2-write(Q). If the graph has a cycle then it is not serializable else it is.
## Transaction Isolation and Atomicity
* if a T1 writes Q and T2 reads Q after that, then if T1 fails then T2 also needs to rollback. This can have a cascading effect other other transactions as well that might be dependent on T2. Thus unless a value is committed, it should not be read by another transaction. This type of read is called dirty read.
## Transaction Isolation Levels
* Serializable isolation level ensures concurrent executions maintain consistency. However it reduces concurrency that might not be enough for some applications. In such cases weaker levels of consistency are used and programmer/application is responsible for ensuring database consistency. These levels of consistency are called isolation levels
* an examples where a different isolation level is desirable is if there is a long running transaction whose result does not need to be precise.
* Serializable: ensures serializable execution. used for complex transactions that require absolute data consistency e.g. a reservation system to prevent overbooking so serializable ensures only one transaction can update the booking at a time.
* Repeatable read: allows only committed data to be read and further requires that betwen two reads of a data item by a transaction no other transaction is allowed to read it (e.g. oracle snapshot too old). I think this is referring to snapshot isolation. Used for stricter level of consistency e.g. banking system doing transactions where database consistency is important to prevent overdraft or inconsistences.
* Read committed: allows only committed data to be read but does not require repeatbale read. new/updated data committed by other transcations can be read. Most common isolation level like a shopping cart where a users wants to see correct items and quantities but not critical if another user adds or removes an item they are viewing, i.e if the inventory becomes 0 then the user wants to see that while adding to cart. A repeatable read might give a wrong impression to user in this case.
* Read uncommitted: allows uncommitted data to be read. An example can be when real time analytics using stock price. It is fine to read uncommitted data in favor of latency.
* All isolation levels disallow dirty writes i.e. to update a data that has already been written by another transaction.
* weaker level of isolation level is chosen for applications for which the inconsistency is not relevant and does not impact its operation.
## Implementation of Isolation Levels
* Locking: transaction locks the data items it accesses. the locks must be held long enough to ensure serializability but not impact performance. DB uses a 2 phase locking strategy where in first phase it only acquires locks but does not release any and in second phase it releases locks but does not acquire any. Locks can be in different modes like shared locks and exclusive locks. The first used for reading and the second for writing. multiple transactions can hold shared locks but only 1 can hold exclusive lock. This allows for concurrent reading but writes are serializable.
* Timestamps: assign each transaction a timestamp, typically when it begins. Each data item has 2 timestamps. read timestamp of the data item holds the largest timestamp of those transactions that read data. write timestamp is the timestamp of the transaction that writes the data.
* Multiple versions and Snapshot Isolation: By maintaining more than 1 version of data it is possible to allow a transaction to read an old version of a data rather than newer version of uncommitted data. In snapshot isolation the transaction is given its own version of the database (repeatable read). Unlike locking attempts to read data never need to wait. Also read only transaction don't need to be aborted as they see a consisted verion of DB and don't see any dirty reads. 
