---
tags:
  - research-paper
site: https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf
title: Google File System
---
## 1. Introduction
#### 1.1 Key points
1. Component failures are the norm rather than the exception.
2. Files are huge by traditional standards. Multi-GB files are common.
3. Most files are mutated by appending new data rather than overwriting existing data.

## 2. Design
### 2.1 Functional and Non Functional requirements
1. Component failures are the norm. The system is built from many inexpensive commodity components that often fail. It must constantly monitor itself and detect, tolerate, and recover promptly from component failures on a routine basis.
2. The system stores a modest number of large files. We expect a few million files, each typically 100 MB or larger in size. Multi-GB files are the common case and should be managed efficiently. Small files must be supported, but we need not optimize for them.
3. Primarily two kinds of reads are done:
	1. Large streaming reads, individual operations typically read hundreds of KBs, more commonly 1 MB or more. Successive operations from the same client often read through a contiguous region of a file.
	2. A small random read typically reads a few KBs at some arbitrary offset. Performance-conscious applications often batch and sort their small reads to advance steadily through the file rather than go back and forth.
4. Primarily write operations are also similar to reads. Small writes at arbitrary positions in a file are supported but do not have to be efficient.
5. The system must efficiently implement well-defined semantics for multiple clients that concurrently append to the same file. Files are often used as producer consumer queues or for many-way merging. Hundreds of producers, running one per machine, will concurrently append to a file. Atomicity with minimal synchronization overhead is essential. The file may be read later, or a consumer may be reading through the file simultaneously.
6. High sustained bandwidth is more important than low latency.

### 2.2 GFS Interface
GFS provides a familiar file system interface, though it does not implement a standard API such as POSIX. Files are organized hierarchically in directories and identified by pathnames. List of operations that are supported:
- *create.*
- *delete.*
- *open.*
- *close.*
- *read.*
- *write files.*
- *snapshot.*
- *record append.*

### 2.3 Architecture
![](https://i.imgur.com/fGD5Ib5.png)

1. GFS cluster consists of single *master* and multiple *chunkservers* and is accessed by multiple clients as shown in the above figure.
2. Files are divided into fixed-size *chunks*. Each chunk is identified by an immutable and globally unique 64 bit *chunk handle* assigned by the master at the time of chunk creation.
3. Chunk servers store chunks on local disks as Linux files and read or write chunk data specified by the chunk handle and byte range. (For reliability, each chunk is replicated on multiple chunkservers.)
4. The master maintains all file system metadata. This includes the namespace, access control info, mapping from files to chunks, and the current location of chunks. It also does periodic garbage collection, chunk migration b/w chunkservers and monitors the state of chunkservers.
5. GFS client code linked into each application implements the file system API and communicates with master and chunkservers to read or write data on behalf of the application.
6. **Caching**: Neither the client nor the chunkserver chaches file data. As often clients stream through huge files caching offers little to no advantage. However clients do cache metadata. Chunkservers need not cache file data as they are stored as local files and Linux's buffer cache already keeps frequently accessed data in memory.

### 2.4 Single Master
Keeping a single node as master vastly simplifies the architecture. However, GFS must minimize its involvement in reads and writes so that it does not become a bottleneck. Hence clients never read or write file data through the master. Instead, a client asks master which chunkservers it should contact. Client caches this information for a limited time and interacts with chunkservers directly for many subsequent operations.

> [!EXAMPLE]- A sample read operation
> - Using the fixed chunk size, the client translates the file name and byte offset specified by the application into a chunk index within the file. 
> - Then, it sends the master a request containing the file name and chunk index. The master replies with corresponding chunk handle and location of the replicas. 
> - The client caches this information using the file name and chunk index as the key. 
> - The client sends request to one of the replicas, most likely the closest one. 
> Note: Further reads of the same chunk require no more client-master interaction until the cached information expires or the file is reopened.

### 2.5 Chunk Size
Each Chunk size can be a file upto 64 MB size.

### 2.6 Metadata
The master stores three major types of metadata. All of this data is stored in master's memory.
1. File and chunk namespaces.
2. Mapping from files to chunks.
3. Location of each chunk's replicas.

The first two types are also kept persistent in local disk via logging mutations to a operation log and replicated on remote machines. This allows to update the master state simply, reliable, and without risking inconsistencies in the event of a master crash. 
The master however doesn't store chunk location information in local disk. Instead, it asks each chunkserver about its chunks at master startup or whenever a chunkserver joins the cluster.

#### 2.6.1 In-Memory Data Structures
Since metadata is stored in memory, master operations are fast. This also helps in periodic scanning type of operations like garbage collection, re-replication in the presence of chunkserver failures and chunk migration to balance load and disk space usage across chunkservers. 
One concern is the entire system is limited by how much memory the master has. This is not a serious limiation as most of the files are much larger in size, meaning most of the chunks would be completely filled, only the last of which may be partially filled.

#### 2.6.2 Chunk Locations
Not persisting the chunk location in local disk of master node has simplified the overall system. This eliminated the problem of keeping the master and chunkserver in sync as chunkservers join and leave the cluster, change names, fail, restart and so. In a cluster with hundreds of servers, these events happen all too often.

#### 2.6.3 Operation Log
The operation log contains a historical record of critical metadata changes. It is central to GFS. It also servers as a logical time line that defines the order of concurrent operations. Files and chunks, as well as their versions are all uniquely and eternally identified by the logical times at which they were created. The master batches several log records together before flushing to disk thereby reducing the impact of flushing and replication on overall system throughput.
The master recovers its file system by replaying the operation log. To minimise the start up time, the operation log must be kept small. The master checkpoints its state whenever the log grows beyond a certain size so that it can recover by loading the latest checkpoint from local disk and replaying only the limited number of logs records that happened after latest checkpoint.

### 2.7 Consistency Model
#### 2.7.1 *Guarantees* by GFS
Did something here.