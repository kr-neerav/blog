+++
title = 'Concurrency Control'
date = 2024-08-20T18:13:28-07:00
draft = false
+++
## Lock based protocols
* when one transaction is accessing a data item it cannot be updated by another transaciton. This is implemented by locks.
* there are 2 modes of lock shared and exclusive.
* a transaction can only proceed with the operation once the concurrency manager grants the locks. a transaction making a lock request cannot execute next action untill concurrency control manager grants the lock. This can result in deadlock if T1 is waiting on A locked by T2 and T2 is waiting on B locked by T1. 
* shared mode lock is compatible with shared mode lock. All other combinations with exclusive mode is non compatible.
* If there are shared locks on a data item and a transaction needs exclusive lock then it needs to wait for all shared locks to be released.
* if we do not use locking or unlock item too soon after reading we may get inconsistent states.
* to identify schedules that are valid the same approach is used of creating graphs and identifying if there is a cycle  in it.
* if a transaction holds a shared lock on data item and another transaction requests a lock on exclusive one, then the second transaction has to wait for first one to complete. In this wait time another transaction can request a shared lock that can be granted. This can lead to a situation where a transaction is starved. This can be prevented by concurrency manager also checking there are no other transctions in queue that are waiting for lock on data item.
### Two phase locking protocal
* Growing Phase: a transaction may obtain locks but may not release any.
* Shrinking Phase: a transaction may release locks but cannot requests any.
* initially tranaction is in growing phase. once the transaction releaes a lock it enters the shrinking phase.
* unlock instruction need not wait to the end of the transaction and can happen in between, just that no locks can be requested after it.
* this results in schedule that are conflict serializable. this is done by ensuring a proper ordering of locks across transactions. the growing and shrinking phase represents a peak that is called lock point. the transactions can be ordered on the lock points.
* two phase locking does not prevent deadlock and can result in cascading failures.
* to prevent canscading failures using a strict two phase locking protocol. in this the locks need to be in 2 phase and exclusive mode locks needs to be held untill transaction completes. This prevents any transaction from reading uncommitted data.
* a variant of this is to obtain locks in shared mode an only upgrade it to exclusive mode when about to write data.
### Implementing the locks
* the lock manager is used to manages locks. it reponds to requests using grants or with messages of rollback in case of deadlocks. unlock messages only require an acknowledgement in response.
* the lock manager uses a hash table of data items. For each data items it maintains a list of transactions that requested locks in the order they were made. It marks the requests that have received locks to be able to differentiate them from the ones that have not. the buckets of the table use chaining if more than 1 data items falls under it.
* It also maintains an index of transactions to be able to efficiently query the locks that are obtained by a transaction.
* when a request arrives it adds a record to the end of the linked list, otherwise it creates a new linked list. it always grants access to data item that is not locked but if its already locks it only grants access if the request is for a compatible locks and all earlier requets have been granted already. Otherwise it waits.
* when it receives an unlock request it deletes the record from the linked list and runs the process mentioned above for next request on linked list.
* if transaction aborts it deletes requests in list that have not yet been granted. Once database takes appropriate action to undo the transaction it releases the locks that were granted to the transaction.
* this algorithm gurantees freedom from starvation, since a request can never be granted if an earlier request is waiting.
## Deadlock Handling
* a set of transactions that are waiting for one another to make progress such that noone can make progress.
* the only remedy is to rollback some of the transaction. The rollback can be partial to the point where the transaction obtained the lock whose release resolves the deadlock.
* there are two methods for dealing with deadlock problem.
* Deadlock Prevention: ensure system will never enter a deadlock state. This is used when the probability that systems enters in a deadlock state is high.
* Deadlock Detection and Recovery: allow system to enter deadlock state and then recover. This is used when the probability that system enters the deadlock state is low. This has additional overhead of maintaining necessary information and execute algorithm to detect deadlock. Additinally it also has cost of recoverying from deadlock.
### Deadlock Prevention
* First approach is to require all locks that a transaction will require to be obtained upfront. So either it obtains lock on all or on none. The disadvantage is that 1) it is often hard to predict at the beginning of transaction what data items need to be locked 2) data item utilization is low since they are locked for a long time.
* Another approach requires imposing an ordering of all data items and to requie transaction to acquire locks in the ordering. However this has the same challenge of not knowing all data items required for transaction at the beginning.
* Preemption: when a transaction Ti requests lock of data item that is held by Tj either it has to wait or Tj is rolled back to allow Ti to proceed. This is decided based on which transaction started earlier based on system timestamp or counter. The transaction that started earlier is given preference.
* wait-die: If Ti started before Tj it is allowed to wait, else it is rolled back (dies).
* wound-wait: If Ti started after Tj it is allowed to wait else Tj is rolled back (wonded) to allow Ti to proceed.
* both the schemes have unnecessary rollbacks.
* Another approach is lock timeouts i.e. each transaction waits for some time to obtain locks and rollsback after the predefined period of time. This will avoid deadlocks as one of the transaction will rollback. however deciding the timeout value is not easy. Too long a timeout results in unnecessary delays when a deadlock has occurred and too short leads to wasted resources (rolled back transactions will run again).
### Deadlock Detection and Recovery
* an algorithm that examines the state of the system to identify if deadlocks have occured is run periodically (detection).
* a directed graph of transactions(nodes) and their dependencies(edges) is maintained.
* when a transaction Ti requests lock on data item that is already locked by Tj an edge is created Tj -> Ti. This edge is removed when transaction Tj releases the lock on data item.
* if there is a cycle in the graph then there is a deadlock.
* in worst case we would need to invoke deadlock detection algorithm for every request for allocation that could not be granted immediately. Otherwise it deadlocks occur rarely, we could run this on a schedule.
* recovery consists of 3 steps
* Selection of a viction: select those transactions that occur minimum cost. its not easy to calculate minimum cost. there are many factors to it e.g. how long the transaction has computed and how much longer it needs, how many data items transctions has used, how many more data items does it need, how many transactions will be involved in the rollback.
* Rollback: one the transaction has been identified, we must identify how far it needs to be rolled back. A simple solution is total rollback and restart. however its  effective to rollback only to the point needed. partial rollback requires system to maintain additional information about the state of all transactions like sequence of lock requets/grants, updates made to data items. In this case the detection algorithm detects the lock that is causing the deadlock instead of transaction. Then rollback is done to that lock. The system must be able to resume the transaction from that point forward.
* Starvation: Based on cost the same transaction might always be picked up for rollback. To avoid this the system only selects a transaction a finite number of times. This is done by including this in the cost factor.
## Multiple Granularity
* In scenario where a transaction needs to lock the entire table or database it will have to lock each item individually which is not performant.
* Locks at different granularity are maintained in a tree format e.g. database -> schema -> file/table -> records.
* Thus a transaction can lock a node in the tree and all its descendents will be assumed to be locked by the transaction.
* If a transaction needs to lock the database it will still need to parse the entire tree to evaluate if it can lock the database. This is not peformant.
* To avoid this as each transaction locks a node in the tree all its ancestors are locked in intention lock modes. Thus if a node is locked in intention lock a transaction has locked one of its descendents and this node cannot be locked completely.
* the nodes can be locked in intention shared mode, intention exclusive mode or shared and intention exclusive mode (if different descendents are locked in shared and exclusive mode)
## Timestamp Based Protocol
* Each transaction assigned a timestamp either a system clock or a counter to identify ordering of transactions. The timestamps determine serializability order.
* To implement this each data item has 2 timestamp values max read timestamp i.e. the timestamp of the latest transaction that read it and max write timestamp i.e. the transaction of the latest trnasaction that execute a write operation on the data item.
* for read (Q) if timestamp of transaction < write timestamp then transaction is looking to read an older value and its rejected else read operation is executed and read timestamp updated for the data item.
* for write(Q) if timestamp of transction is < read timestamp then there are transactions that depend on the current value of Q and hence write is rejected. If timestamp of transaction is < the write timestamp the transaction is planning to write a stale value and its rejected. otherwise the write transaction is executed.
* if a transaction is rolled back it is assigned a new timestamp and restarted.
* the timestamp ordering protocol ensures conflict serializability as conflicting operations are processed in timestamp order.
* it also ensures freedom from deadlock as no transactions wait on another.
* it can lead to starvation of long transactions when it conflicts with shorter transaction. Thus after a few restarts the short transactions need to be paused to allow long transaction to complete.
* the protocol can generate schedules that are non recoverable.
* to ensure recoverability and cascadelessness all writes must be peformed together at the end of the transaction.
### Thomas Write Rules
* <revisit>
## Validation Based Protocols
* In systems where majority of the transactions are ready only the rate of conflict is low. a concurrency control schemes imposes overhead on code execution and an alternate scheme with less overhead might be beneficial.
* <revisit>
## Snapshot Isolation
* each transaction views its own snapshot of database as of the starting of the transactions. read transction are never blocked
