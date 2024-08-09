+++
title = 'Distributed Databases'
date = 2024-07-30T21:46:36-07:00
draft = true
+++

## Notes
### Homogeneous and Heterogeneous Databases
* Parallel database have tightly coupled processors while distributed database is comprised of loosely coupled sites that share no physical components.
* A transaction can span more than 1 sites.
* The distribution of data is the cause of many difficulties.
* Homogeneous systems use the same DBMS software, are aware of one another and agree to cooperate in processing requests. The local sites surrender portion of their autonomy like right to change schema or DBMS software used.
* Heterogeneous systems are totally decoupled with each other, can use different DBMS software and can have different schema as well. The schema difference is a major challenge.
### Distributed Data Storage
* There are 2 approaches to store a table in distributed db. either replicate entire table at multiple sites or fragment the table and store each fragment at a different site.
* Often fragmentation and replication are compbined.
#### Replication
* Replication can be done either at a few sites or at all sites.
* It increases availability, parallelism for read operations as several sites are available to participate in the transaction, minimize data movement as data might be present locally where transaction is being processed.
* The disadvantage is that updates are costly due to increased overheard to update all replicas.
* Concorrency control is harder. We can simplify it by choosing 1 copy as primary and other as replicas. This makes concurrency control similar to a centralized system.
#### Fragmentation
* Two ways to fragment a table, horizontal fragmentation and vertical fragmentation.
* Horizontal fragmentation is used to keep tuples at a site where its most used to minimize data movement.
* In vertical fragmentation columns are split. A way to ensure tuple can be reconstructed is to include primary key for the tuple in each split or a system identifier like tuple-id.
* Both type of fragmentation can be combined.
#### Transparency
* User of the system does not need to be aware about the internals i.e. data is fragmented, replicated and its location. They should see a unique logical view of the data.
* table, fragments and replicas must have unique name. It is easier to do in a centralized system. In a distributed system this is achived by requiring all names to be registered to a central name server.
* The name server is also used to locate a data item given its name.
* The challenges with name server are that it is a single point of failure and can also become a bottleneck for all transactions that require search by name.
* An alternate approach is to prefix the name with site details. This ensures names will be globally unique and does not require a central service. However this breaks location transparency as the user now needs to be aware of the prefix and the file name to query.
* An alternate approach is that database uses alias and each site is responsible to map the alias to the file name (by keeping a local mapping). This ensures local transparency.
### Distributed Transactions
* transactions on local copy are standard as defined for centralized system.
* transactions involving multiple sites are more complex as they need to take into account failure at any one of the sites as well.
#### System Structure
* There is a transaction manager at each site responsible for managing local transactions just like in a centralized database.
* There is a transaction coordinator at each site to coordinate transactions across sites. It is responsible to execute the transactions at multiple sites. 
#### System Failure Mode
* A distributed system may fail for reason similar to ce3ntralized system like software errors, hardware errors, disk crashes.
* There are additional failure mode like failure of a site, loss of message, failure of communication link, network partition.
* Loss of message is addressed by using TCP protocal which addresses loss of message by requesting it again.
* if communication link is broken then it is addressed by finding an alternate link.
* if there are no alternate routes then it is called network partition. It can even apply to a sigle node apart from cluster of nodes.
### Commit Protocols
* It is to ensure either all sites commit transaction or all abort.
#### Two Phase Commit
* C = Transaction Coordinator; T = Transaction; S = Site at which transaction originated
* When all sites confim to S that the transaction has been execute the process of commit begins.
* C adds a <prepare T> entry to the durable log and sends out notification to all required sites to do the same.
* On receiving such a message the transaction manager at each sites determines if it can commit the transaction. It sends a <ready T> or <no T> entry to its durable log and sends the same to C.
* If any of the site sent <no T> then transaction is aborted.
* If the site send a <ready T> response then it must maintain all locks for the duration of the commit.
* If C did not hear from some sites after a specified time period it can decide to abort the transaction.
* Depending on the decision by C a <commit T> or <abort T> entry is added to the log and shared with other sites.
* When the site receives this message it logs it in the log and commit/aborts the transaction.
* At the end of the phase each site sends and acknowledgement message to C acknowledging the action which is <complete T>.
* If C does not hear back on <prepare T> from a given site it aborts the transaction.
* If C hears back on <prepare T> but does not hear back on acknowledge T it does nothing and proceeds with the transaction. The the site records it is their responsibility to identify the state of transaction and take appropriate local action.
* When site recorvers it checks its log. If its find <commit T> it redo the transactions, if its find <abort T> it undo the transaction. If it finds <ready T> it tries to connect with C to find the status of the transaciton. If C is not available it broadcasts to all sites to understand the state of transaction. Once it finds the state out it applies undo or redo. If it does not find the state out it periodically checks for the status of transaction with all sites.
* If C fails. If each site finds an <abort T> or <commit T> in its log it knows a decision was made by C and implements it.
* If site does not find <ready T> then C could not have made a commit decision and safely aborts the transaction.
* If it finds <ready T> then it does not know the decision made. it checks with C to understand the decision and take that action. If C is not responding other sites wait for it to recover. In the mean time it blocks the resources it had locked for the transaction which makes it unavailable for other transactions. This situation is called a bllocking problem. This blocking is a major drawback of 2 phase commit
* Incase of network partition if all sites are in the same partition the transaction proceeds normally. If they are on different partitions then the above rules are followed.
* Recovery and Concurrency Control: when recovering the sites makes decision on all pending transactions. It cannot be usable till those decisions are made. Since it takes time and C might not be available recovery can be long. To expedit the recovery the log also include details on write logs i.e. <ready T, L>. Thus the site can recover quickly by reacquiring the locks. Once locks are in place concurrent transctions can proceed if they don't have a conflict with the locks. For concurrent transactions that have a conflict with the locks they need to wait till that transaction is resolved.
* Three Phase Commit: Instead of having just 1 C, multiple sites have this information. They elect a C. Thus is C fails another C is selected and resources are not blocked for long.
* Persistent messaging: 2 phase commits blocks resources. There are scenarios where blocking is not acceptable. Persistent messaging is used in that case. Persistent messaging is guarantee that messages will be delivered once. E.g. in banking checks are used to transfer money, however account balance are not locked. Instead it is guaranteed checks will not be lost and will not be duplicated. This can ensure the transaction completes asynchronously.
* Error handling is more complex as if a receiving site is down (like account closed) the money needs to be restored back to original account. Such error handling will depend on application as well and can't just dependent on database capabilities. This is more complex than 2 phase commit where the responsibility of commit is only with DB and not with application.
* Implementation of persistent messaging.
* Sending Site Protocol: The transaction writes a persistent message to a database, instead of delivering to the receiver and provided a unique id. There is a message deliver component that reads the database for new records and attempts to deliver to the destinaton. It continues to try till it receives an acknowledgement of delivery. There might be timetout configured. In this approach message is only written to database when the transaction completes at sending site.
* Receiving Site Protocal: The receiver runs the transaction and stores the message in a table if its not already present. After transaction complete or if the message was already present in the table the receiver sends and acknowledgement back. Sending acknowledge before the transaction completes is not safe as system failure might result in loss of message. Ideally message should not be deleted from receiver's table to avoid executing transaction more than once. However this can lead to indefinite size of the table. To avoid this a cutoff period is identified, persistent messages with timestamps older than that are discarded and a similar cutoff retention is also applied on the receiver table.
### Concurrency control in distributed databases
#### Locking Protocols
* The locking protocols can be used for distributed environments as well with the change that lock manager needs to deal with replicated data.
* Single Lock Manager: A single site acts as a lock manager with all sites requests it for locks. This simple to implement as its similar to centralized database and has simple deadlock handling as all locking  information present at a single site. The disadvantage is that it is a single point of failure and also bottleneck for performance.
* Distributed Lock Manager: each sites maintains a local lock manager to lock items on that site. Its simple to implement. However deadlock is more complex since locking requests are no longer present at a single site.
* Replication is handling by using concept of primary and replica. The primary site can handle locks using local lock manager. However if primary site fails the data is not available for locking even when its available to read from the replicas.
* Majority Protocol: For N copies of data it requires lock on atleast N/2+1 copies. This is helpful when there are site failures. The disadvantages are more complex implementation involving multiple sites and scenario of deadlock if different transactions have locked N/2 sites and both are waiting on atleast 1 more site to be locked.
* Biased Protocol: It has different mechanism for read and write operations. For read operation it requests shared locks that can be obtained from the lock manager at site. For write operation it obtains locks from all sites. Thus reads are relatively faster than writes. Helpful for most scenarios as reads are generally more than writes.
* Quorum protocol: each site is assigned weights and read/write operations has minimum threashold of locks required, r for reads and w for writes. For read operations sum of weights for sites should be >= r and for writes it should be >= w. This helps to control read/wirte performance independently. Additionally some sites that are less likely to fail can be assigned higher weights thus reducing the number of sites that need to be locked.
#### Timestamping
* each transaction is given a unique timestamp that the system uses to decide the serialization order.
* There are 2 ways to generate unique timestamp centralized and distributed.
* In centralized scheme a single site distributes the timestamp. it can be a logical counter or its own timestamp.
* In decentralized schema the site identifier is added as a suffix to the local logical counter or timestamp. To esnsure one site with too many transactions does not influence the serialization order all clocks needs to failure increment their counter. This is done by ensuring the logical counter at each site is updated when it redeives a transaction with timestamp x.
#### Replication with weak degrees of consistency
* In master slave replication read transactions do not request locks at replicas. But they need to be provided a consistent view of the database which might be outdated i.e. a transaction must see all state of data till transaction X where transactions are applied in a serialized manner.
* System propogates updates to primary ASAP or at periodic interval. System can use 2 phase commit to update all replica or biased protocol whre writes updates all replicas and reads take place at any 1 replica.
* Another option is to update at 1 site and it lazy propogates to other sites. This allows transaction to proceed even if some sites are ofline but the trade off is consistency.
* Update at 1 site are translated to primary. The rest of the replicas are updated by the primary using lazy propogation.
*  Updates take place at any site and are propogated from there. This can cause conflicts if different replicas are updated by different transactions at the same time.
#### Deadlock handling
* revisit
### Availability
* one of the goals of using distributed databases is high availablity. for this it must detect failures and reconfigure system so the computation may continue and repair when the failed site recovers.
* different type of failures are handled differently e.g. message loss by retransmission.
* repeated transmission of a message without an acknolwedgement means a link failure. the system must find an alternate route. failure to find an alternate route means network partition.
* system can detect a failure but not the type of failure like site failure vs network partition. failure to reach out a node can be due to node failure or network partition. multiple links are connected to a site to avoid network partition but in rare scenarios  all links can fail.
* if a site fails that had an active transation then the transaction must be aborted as it might be holding resources at other sites. Incase data is replicated transaction may be allowed to proceed and when the failed site recovers it need to catch up on all the updates to its data.
* if replicated data is stored at the failed site then catalog should be updated so queries do not reference copy of the failed site. when the site rejoins care needs to be taken that the data is upto date.
* if failed site is a central service like deadlock detector, name server etc then an election must be held to elect a new leader.
#### Majority based approach
* each object is stored with a version number which is incremented each time a write transaction updates the data.
* write transaction only proceeds when it receives locks on atleast half the replicas.
* read operations look at all replicas on which a lock was obtained (mabe shared lock) and read the value from the object with highest version number. Sometimes they update the replicas with lower version numbers as well.
* writes read all replicas to get the latest value of object i.e. the one with highest version number.
* reintegration is trivial as nothing needs to be done as reads and writes using multiple sites to find the latest value.
#### Read One, Write all available approach
* special case where all sites have unit weight, r = 1 and w = all replicas.
* The challenge is that if any site is down no write operation can proceed. The good part is that no version needs to be maintained on objects.
* An alternate approach is to proceed writes on all available sites. This can have challenges as writes are not updated on failed sites and when they recover they do not know that they need to reintegrate in the system.
#### Site Reintegration
* when a failed site recovers it must initiate a procedure to updates its data to reflect the changes made when it was down.
* reintegration is challenging as the system is constantly updating, even during reintegration phase.
* one approach is to halt the entire system when the site reintegration takes place but this causes disruption. There are algorithms created for it.
#### Coordinator Selection
* many algorithms need a central coordinator. Incase a coordinator fails system must identify a new coordinator to continue.
* Backup coordinator is when another site acts as a back of the current coordinator, receives all updates to update the internal state, but does not take any action. Incase it detects failure of coordinator it acts as a coordinator.
* The benefit is that transactions can contiue as it has all the information about the transactions. However this requires the overhead of maintaining 2 coordinators and both need to communicate regularly to ensure activities are synchronized.
* If there is no backup  cooridnator the system elects a new coordinator which has to catch up on the transaction by communicating with all sites. Some details that were only present with the coordinator will be lost and some transactions will be aborted due to lack of information.
* Bully algorithm: if a site sends message to the coordinator and does not receives a response it assumes the coordinator has failed and tries to elect itself as the coordinator. Each site has an identification number and the coordinator will always be the site with highest identification number. The site sends it identification number asking if there is a site with higher identification number. If it does not receive a response it assumes all sites with higher identification number hvae failed and it elects itself as the leader and notifies all sites. If it does receives response from a site with higher identification number it waits for time T to receive a message that a new site is the coordinator. If it does not receive message in time T it assumes that site has failed and starts the process again.
* if a failed site recovers it tries to identify sites with identification numbers greater than its. If it does not receives a response, it forces all sites to choose it as the coordinator event if the coordinator is already present with a lower identification number. If it for this reason that the it is called bully algorithm.
#### Trading off consistency for availability
* In a distributed database network partitions cannot be prevented. Thus either availability or consistency needs to be sacrificed. Algorithms till now sacrifice availability for consistency. But the system might become unresponsive if w or r cannot be achived due to network partitions.
* updates should be allowed even in the case of partitions.
* eventual consistency requires inconsistent copies to be identified, propogating updates. inconsistent updates needs to be merged in a meaningful way. This cannot be done by the database and needs to be handled by application.
### Distributed Query Processing
* In centralized system query optimization is done by focusing on cost of disk access. In distributed system it also needs to include network cost, parallelism. Its a tradeoff between disk and network cost.
* The table is replicated and fragmented. If the table is not fragmented then the process is relatively simpler.
* Joining 3 tables that are not fragmented but replicated at different sites. No one strategy works all the time. Factors to consider are volume of data at each site, cost of transmission and speed of processing at each site. 1) ship 2 tables to 1 site and perform join locally. if there were indices that could have helped with join then those are no longer useful and a most costly join strategy might be selected. 2) Ship 1 relation to 2nd site and do join then ship the temp relation to 3rd site. This might result in additional network cost as join might result in denormalization of one of the tables.
* Semijoin strategy: if only a small number of tuples from a relation will participate in join it is costly to ship the whole relation for join. Thus a semi join approach is used. In this the join columns from r1 are first shipped to r2 and join performed locally. then the joined dataset is shipped back for joining with r1. This reduces cost of shipping tuples from r2 that will not participate in join but has additional cost of shipping join column from r1. This is used when there is join of relations that involve filter as well. For multi table join the same strategy is repeated.
* Parallilize the join in multi table join like r1 x r2 x r3 x r4. there are many ways in which a combination can be chosen to parallilize 4C2.
### Data Store systems in cloud
* high scalability requirements require data to be partitioned across 1000s of processors.
#### Data Representation
* an object like user profile contains variety of attributes and there are frequent additions to the attributes stored in profile. Some attributes contain complex data like struct or array. A relational representation is not sufficient. JSON or XML is used.
* These applications do not need extensive query support as primary access pattern is value for a key.The applications that require join do it through materiazlization. Instead of joining on the fly updates to one table are propogated to other table as messages and hence all data is available at one place. e.g. for news feed whenever a user aposts something a message is sent to news feed of all their friends which are updated. then when the friend logs in they can see what their friends have posted in a single place.
* Thus cloud data store have 2 basic functions put(key,value) and get(key).
* Big table data model can support large number of arbitary columns. The key in case case is (key value , attribute name). To find all attributes for a record a range query or prefix match query is used with key value. The system stores entries sorted by key so all attributes are clustered together for faster retreival.
* Partitioning and retreiving data: the partition function cannot be decided ahead of time. Also if load increases more servers need to e added and each server should be able to take parts of the load. To solve for this data storage system partition data into relatively small units. This is done on the search key. two approaches used. either range partitioning directly on search key or range partitioning done on the hash of search key. This partitions into units happens dynamically. As table grows it is broken into smaller parts or if the load on the table is too much unit is broken up into parts. Number of units is greater than number of sites. The hash function maps record to unit and another mapping function maps unit to site. Protocols need to be created to update mapping file when a unit is split.  
* Transactions and replication: cloud data storage systems do not support two phase commit as the cost is too high and it can lead to blocking in event of failures. Thus it cannot support secondary indexes on different key as the index record to be updated might be in a different site, requiring two phase commit.
* Updates are performed by compare and set i.e. the current value is checked to verify it has not updated during the transaction. sometimes version number for the tuple is also used.
* Replication of units is done to ensure data is available incase of site failures. if a site fails the units on the sites need to be reassigned to a different site. 
### Directory Systems
* Why does Directory Access Protocol exist instead of using a database over JDBC? first DAP are simplified protocols that cater to limited types of access to data. Second DAP is a mechanism to store data in a  hierarchical fashion like in multiple servers arranged in a hierarchy.
* LDAP: Lightweight Directory Access Protocol
* 
