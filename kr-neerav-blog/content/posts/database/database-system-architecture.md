+++
title = 'Database System Architecture'
date = 2024-08-24T20:15:15-07:00
draft = true
+++
## Server System Architecture
* they can be classified as transaction servers and data servers.
* Transaction Servers or Query Servers: Clients ship instructions in the form of transactions which are executed on server and result sent back. Requests are usually specified by SQL. This is widely adopted. 
* Data Servers: allows clients to make requests to read or update data in units such as file or pages e.g. file servers.
* ### Transaction Servers
* consists of multiple processes accessing data in shared memory. refer to page 774 for diagram.
* Server Process: this process receives use queries (transactions), execute them and send the results back. queries are submittted via JDBC, ODBC or similar protocol. some databases uses a process for each user sessions while some use multiple user sessions in each process but each session has a different thread.
* Lock Manager Process: implements lock manager. It grants, releases and detects deadlock
* Database Writer Process: output modified buffer blocks to disk on continuous basis. runs in the background.
* Log Writer Process: ouputs log records from log record buffer to stable storage. server process simply add log records to log buffer.
* Checkpoint Process: performs periodic checkpoints.
* Process Monitor Process: monitors other processes and if any of them fails it takes recovery actions for the process such as aborting any transactions being executed by failed process and restarting the process.
* Shared Memory: it consists of all data like buffer pool, lock table, log buffer, cached query plans (which can be resued if same query is submitted again)
* all database process can access data in shared memory. since multiple process can update the data in memory there must be a mechanism to ensure data is modified by at most 1 process at a time and no process is reading the data structure while its written. OS provides functionality for this.
### Data Servers
* used in local area networks with high speed connections between client and servers, client machines are compareable in processing power to server machine and tasks to be executed are computation intensive. In this environment it makes sense to ship data to client machines to perform all processing on client and ship data back to server. This is populat in object oriented database systems (my observation also seen in object storage or block storage)
* Page shipping: unit of communication is page instead of tuple. If it was tuple the overhead of message passing is high compared to the amount of data transmitted (when compared to shared memory). It is also considered as a form of pre-fetching as other tuples in the page might also be used in processing later.
* Adaptive lock granularity: When page is shipped to client it is locked by the server. If the client is only processing a tuple the entire page is locked resulting in blocking other processes. client can either update the server back on the lock it requires within the page, thus unblocking the rest of the tuples in the page for other transactions.
* Data caching: data shipped to the client is cached even after transction completes. successive transactions may be able to make us of cache data. cache coherency is an issue where the data in cache needs to be validated that it is the same as data on server. a message is exchanged with the server of the validity of data and requesting lock on it.
* Lock caching: if data is mostly partitioned among client with each rarely accessing data processed by others, locks can also be cached at client machine. however the server must take the responsibility of calling back conflicting locks.
## Parallel Systems
* Improve processing and I/O by using mulitple processors and disks in parallel.
* supports querying large amount of data or large number of transactions per second.
* many operations are performed simultaneously as opposed to serial processing in which steps are computed sequentially.
* a coarse grain parallel machine consists of a small number of powerful processors.
* a massively parallel or fine grained paralled machines uses 1000s of smaller processors.
* measures of performance of database 1) throughput, number of tasks completed in a given interval 2) response time, amount of time it takes to complete a single task.
* a system processing large number of small transactions can improve throughput by processing many in parallel. a system that process large transactions can improve response time as well as throughput by performing subtasks for each transaction in parallel.
### Speed up and Scale up
* speed up: running a given task in less time by increasing degree of parallelism
* scale up: handling larger tasks by increasing degrees of parallelism.
* scale up more important metrics for measuring efficiency of parallel systems.
* factorst that work against efficient parallel system 1) start up costs 2) interference 3) skew
* start up costs: creating a process has a cost. if 1000s of processes need to be started the start up time may overshadow the actual processing time affecting speedu up adversely.
* interference: processes access shared resources and each new process competes with existing resources for commonly held resources like system bus, shared disks, locks etc. both speed up and scale up are impacted by this.
* skew: the service time for the single slowerst step will determine the service time for the task as a whole. it if often difficult to divide a task into equal sized parts and they way sizes are distributed are often skewed.
### Interconnection Networks
* components (processors, memory, disks) communicate with each other via an intercommunication network. page 781 for digram
* Bus: all system components send on and receive data from a single communication bus. This could be an ethernet. It works well for small number of processors however does not scale well with increasing parallelism, since bus can handle communication from 1 components at at time.
* Mesh: components are nodes in a grid and each components connects to all its adjacent components on the grid. Nodes that are not directly connected can communicate with one another by routing messages via  sequence of intermediate nodes that are connected to each other. As the number of components grows, the number of links also grows, thus increasing the communication capacity of the mesh.
* Hypercube: More efficient verion of mesh where nodes and connected in a specific way to reduce the communication delays.
### Parallel Database Architectures
#### Shared Memory
* Processor and disks have access to a common memory typically via a bus or through an interconnection network. 
* extermely efficient communication between processors via shared memory for reading data or passing messages.
* Does not scale beyond 32 or 64 processors because the bus or network becomes bottleneck.
* have very large memory caches at each processor to avoid referencing shared memory when possible. this requires caches to be kept coherent i.e writes to data item in 1 cache invalidates data item in all other caches which either needs to be updated or removed.
* maintaining cache coherency becomes an increasing overhead as number of processors increase.
#### Shared Disk
* all processors can access all disks directly via network but processors have private memories.
* since each processor has its own memory, memory bus is not a bottleneck.
* provides a degree of fault tolerance. i.e. if a processor fails other processor can take up the tasks.
* Disk is made fault tolerant by using RAID architecture.
* disk subsystem is the new bottleneck especially in situations when database makes large number of requests to the disk.
* this can scale to somewhat larger number of processors but communication across processors is slower as it goes through communication network.
#### Shared Nothing
* nodes connected to each other through a high speed communication network
* node acts as a server for the data on its disk.
* local disk references are serviced by local disks at each processor.
* in this system compute is pushed to data instead of moving data. Only data that requires access to non local disks is pushed on the communication network.
* more scalable and can easily support large number of processors.
* the main cost of the system is of communication and of non local disk access.
## Distributed Systems
* nodes vary in size function, do not share anything and interact using network or internet.
* difference from parallel systems: distributed systems are geographically separated and have a slower interconnection. also there is a difference between local and global transactions. local transaction access data from a node while global one accesses data from other nodes as well.
* reasons for building distributed systems
* sharing data: users at one site are able to share data with other sites.
* autonomy: each site is able to retain control over the data stored locally. there is a global database administrator but responsibilities are also delegated to local sites.
* availability: if one site fails other sites may be able to continue to operate. if data items are replicated in several sites transaction can find the data item in other sites. failure of site does not imply shutdown of system.
### Implementation issues
* atomicity of transaction is an important issue. a transaction may commit at one site and abort on another. To avoid this it uses a two phase commit protocol.
* 2 phase commit: each site/node enteres a state where it partically commits the transaction and leaves the commit decision to a single coordinator site/node. Every node must follow the decision of the coordinate node, whether to commit or rollback the transaction.
* concurrency control is another issue. since transaction may access data at several nodes, transaction managers at several nodes need to coordinate together to implement concurrency control.
* locks on local data item are simple but can result in deadlocks for items locked from other sites. deadlock detection needs to be carried out across sites.
* failures are more common in distributed systems as its not just the node but the communication links may also fail.
* replication of data items further complicates concurrency control.
* 
