# Distributed File Systems (DFS), HDFS (Hadoop DFS) Architecture and Scalability Problems

## Distributed File Systems (DFS)

One node will get out of service in three years on average. 
With a cluster of 1,000 nodes you will get one pillar each day. 
You would like to store your data in a reliable way → 
stumbling block of DFS lies

![test](/Images/1_Big_Data_Essentials/Week_1/DFS.png)

### GFS (Google File System)

* Components failures are a norm ( → replication / dublication)
* Even space utilisation (files a split into blocks of fixed size (around 100MB))
* Write-once-read-many (simplifies API and internal implementation of DFS)

![test](/Images/1_Big_Data_Essentials/Week_1/GFS.png)

Replication: S and B are split into equal sized blocks and distributed over different machines with replication. 

Chunksever = Datanodes = Storage Machines

### HDFS (Hadoop File System)

![test](/Images/1_Big_Data_Essentials/Week_1/HDFS.png)

* Namenode (HDFS) = masternode (GFS)
* Command line interface
(No need to write code to access data)
* Binary RPC protocol with option to access data via HTTP protocol

#### How to read files from HDFS

![test](/Images/1_Big_Data_Essentials/Week_1/HDFS_read.png)

Get data from the closest machine → data center topology

1) Request namenode to get information about file blocks’ location
2) These blocks are distributed over different machines, but all of this complexity is hidden behind HDFS API.

#### Data Center topology

![test](/Images/1_Big_Data_Essentials/Week_1/DC_topology.png)

* Data on same machine: d = 0
* Data node in same rack: d = 2
* Data node in another rack: d = 4
* Data node in another data center: d = 6 (delivery overhead)

#### How to write files to HDFS

![test](/Images/1_Big_Data_Essentials/Week_1/HDFS_write.png)

1) Request a namenode via RPC
2) Namenode checks your rights and naming conflicts
3) HDFS client requests a list of datanotes to put a fraction of blocks of the file
4) These datanodes form a pipeline, HDFS client sends packets of data to the closest datanode.
5) The later one transfers copies of packets through a datanode pipeline.
6) As soons as packet is stored on all of the datanodes, datanodes sent acknowledgment packets back.
7) If something goes wrong, HDFS is closes the datanode pipeline, marks the misbehaving datanode bad, and request a replacement from the namenode for the bad node → new datanode pipeline will be organized.

#### What happens with failure blocks?

![test](/Images/1_Big_Data_Essentials/Week_1/Failure_blocks.png)

1) Datanode saves the state of a machine for each block.
2) Wherever a datanode recovers from its own failure, or failures of other datanode in the pipeline, you can be sure that all necessary replicas will be recovered and unnecessary ones will be removed.

#### Summary
* You can explain what vertical and horizontal scaling is
* You can list server roles in HDFS (datanode and namenode)
* Explain how topology affects replica statement
* What is a chunk/block size of data and how does it help to balance cluster loads
* Explain how HDFS client reads and writes data

## Block and Replica States, Recovery Process

#### Block and Replica
![test](/Images/1_Big_Data_Essentials/Week_1/Replica.png)

* Replica is a physical data storage on a data node. 
There are usually several replicas with the same content on different data nodes. 
* Block is a meta-information storage on a name node and provides information about 
replica's locations and their states. 
* Both replica and block have their own states. 

### Datanode replica's states

![test](/Images/1_Big_Data_Essentials/Week_1/Sim_Replica_1.png)

#### Finalized
* If replica is in a finalized state then it means that the content of this replica is frozen. 
* The latter means that meta-information for this block on name node is aligned with all the corresponding replica's states and data. 
    * For instance you can safely read data from any data node and you will get exactly the same content. 
    * This property preserves read consistency.
* Each block of data has a version number called Generation Stamp or GS. 
For finalized replicas, you have a guarantee that all of them have the same GS 
number which can only increase over time. 
It happens during error recovery process or during data appending to a block.

#### Replica being written (RBW)
* It is the state of the last block of an open file or a file which was reopened for appending. 
* During this state different data nodes can return to use a different set of bytes. In short, bytes that are acknowledged by the downstream data nodes in a pipeline are visible for a reader of this replica. 
* Moreover, data node on disk data and name node meta-information may not match during this state. 
    * In case of any failure data node will try to preserve as many bytes as possible. 
    * It is a design goal called data durability.

#### Replica Waiting to be Recovered (RWR)
* Is a state of all being written replicas after data node failure and recovery. 
    * For instance, after a system reboot or after a BSOD, which are quite likely from a programming point of view. 
* RWR replicas will not be in any data node pipeline and therefore will not receive any new data packets. 
* So they either become outdated and should be discarded, or they will participate in a special recovery process called a lease recovery if the client also dies.

#### Replica Under Recovery (RUR)
* In case of HDFS client lease expiration, replica transition to a RUR state. 
* Lease expiration usually happens during the client's site failure. 
As data grows and different nodes are added or removed from a cluster, data can become unevenly distributed over the cluster nodes. 
* A Hadoop administrator can spawn a process of data rebalancing or a data engineer can request increasing of the replication factor of data for the sake of durability. 

#### Temporary
* In these cases new generated replicas will be in a state called temporary. 
* It is pretty much the same state as RBW except the fact that this data is not visible to user unless finalized. 
* In case of failure, the whole chunk of data is removed without any intermediate recovery state.

### Namenode replica's states

![test](/Images/1_Big_Data_Essentials/Week_1/Sim_Replica_2.png)

* In addition to the replica transition table, a name node block has its own collection of states and transitions. 
* Different from data node replica states, a block state is stored in memory, it doesn't persist on any disk. 

![test](/Images/1_Big_Data_Essentials/Week_1/Sim_Replica_3.png)

#### Under Construction
* As soon as a user opens a file for writing, name node creates the corresponding block with the under_construction state. 
* When a user opens a file for append name node also transition this block to the state under_construction. 
* It is always the last block of a file, it's length and generation stamp are mutable. 
* Name node block keeps track of right pipeline. 
    * It means that it contains information about all RBW and RWR replicas. 
    * It is quite vindictive and watches every step.

#### Under Recovery
* Replicas transitions from RWR to recovery RUR state when the client dies. 
* Even more generally it happens when a client's lease expires. 
* Consequently, the corresponding block transitions from under_construction to under_recovery state.

#### Committed
* The under_construction block transitions to a committed state when a client successfully requests name node to close a file or to create a new consecutive block of data. 
* The committed state means that there are already some finalized replicas but not all of them. 
* For this reason in order to serve a read request, the committed block needs to keep track of RBW replicas, until all the replicas are transitioned to the finalized state and HDFS client will be able to close the file. 
* It has to retry it's requests.

#### Complete
* Final complete state of a block is a state where all the replicas are in the finalized state and therefore they have identical visible length and generation stamps. 
* Only when all the blocks of a file are complete the file can be closed. 
* In case of name node restart, it has to restore the open file state. 
* All the blocks of the un-closed file are loaded as complete except the last block which is loaded as under_construction.

#### Recovery procedures
* Then recovery procedures will start to work. 
* There are several types of them:
    * replica recovery
    * block recovery
    * lease recovery
    * pipeline recovery

#### Questions:
Q: Could we have “finalized” replicas with different visible lengths or generation stamps?

A: No. For finalized replicas you have a guarantee that all of them have the same GS number and visible lengths.

#### Block Recovery
![test](/Images/1_Big_Data_Essentials/Week_1/Block_recov.png)

* During the block recovery process, namenode has to ensure that all of the corresponding replicas of a block will transition to a common state, logically and physically. 
* By physically, I mean that all the corresponding replicas should have the same disk content. 
* Namenode choses a primary datanode called PD in a designed document. 
    * Obviously PD should contain a replica for the target block. 
    * PD request from a namenode and new generation stamp, information and location of other replicas for recovery process. 
    * PD contacts each relevant datanode to participate in the replica recovery process.
    * Replica recovery process includes aborting active clients writing to a replica. 
    * Aborting the previous replica or block recovery process, and participating in final replica size agreement process. 
    * During this phase, all the necessary information or data is propagated through the pipelines. 
    * As the last step, PD notifies namenode about the result, success or failure. 
    * In case of failure, namenode could retry block recovery process. 

#### Lease Recovery
![test](/Images/1_Big_Data_Essentials/Week_1/Lease_recov_1.png)

* Block recovery process could happen only as a part of a lease recovery process. 
* Lease manager manages all the leases at the namenode. 
* HDFS clients request a lease every time they would like to write or append to a file. 
* Lease manager maintains a soft and a hard limit. 
    * If a current lease holder doesn't renew its lease during the soft limit timeout, then another client will be able to take over this lease. 
    * In this case and in this case of reaching a hard limit, the process of lease recovery will begin. 
    * This process is necessary to close open files for the sake of the client. 
    * During this process there are several guarantees to be achieved. 
        * The first one is concurrency control, even if the client is alive it won't be able to write data to a file. 
        * The second one is consistency guarantee. All replicas should draw back to a consistent state to have the same on this data and generations temp

![test](/Images/1_Big_Data_Essentials/Week_1/Lease_recov_2.png)

* Lease recovery starts with lease renew
* Of course, new files lease holder should have superpower includes ability to take ownership of any other user's lease. 
* In our case, the name of the superuser is dfs. 
* Therefore, all of the other clients requests such as get new generation stamp, get new block, close file from other clients to this path will be rejected. 
* Namenode gets the list of datanodes which contain the last block of a file, assigns a primary datanode and starts a block recovery process. 
* As soon as block recovery process finishes, namenode is notified by PD about the outcome. 
* Updates block in for and removes the list from the file. 

#### Pipeline Recovery
![test](/Images/1_Big_Data_Essentials/Week_1/Pipeline_recov_1.png)

* When you write to an HDFS file, HDFS client writes data block by block. 
* Each block is constructed through a write pipeline. 
* HDFS client breaks down block into pieces called packets. 
    * These packets are propagated to the datanodes through the pipeline. 
    * As illustrated in this picture, there are always three stages 
        * pipeline setup
        * data streaming
        * close 
    * Both lines on this image represent data packets 
    * Dotted lines represent acknowledgement messages
    * Regular lines are used to represent control messages
* During the pipeline's setup stage, the client sends a setup message down through the pipeline. 
    * Each datanode opens a replica for writing and sends a message back upstream the pipeline. 
* Data streaming stage is defined by time range from T1 to T2. Where T1 is the time when a client receives the acknowledgement method for a setup stage. And T2 is the time when a client receives the acknowledgement method for all the blog packets. 
    * During the data streaming stage data is buffered on the client's site to form a packet, then propagated through the data pipeline. 
    * Next packet can be sent even before the acknowledgement of the previous packet is received.

Q: What is a replica’s state in this case? 

A. RBW (If there are no failures, then it is an RBW replicas state (the process is already started, but not finalized, so other options are not possible))

![test](/Images/1_Big_Data_Essentials/Week_1/Pipeline_recov_2.png)

* As you can spot into this image, there are some specific data packets called flash. 
    * They are synchronous packet and used as a synchronization point for the datanode write pipeline. 
* The final stage is close, which is used to finalize replicas, and shut down the pipeline. 
    * All the datanodes in the pipeline change the replica states to the finalized. 
    * Before they state to a namenode and send the acknowledgement method upstream. 
    * In case of datanode pipeline recovery can be initiated during each of these stages. 
    * If you are writing to a new file, and a failure happens during the setup stage. They easily abandoned datanode pipeline, and and request a new one from scratch. 
    * If datanode is not able to continue process packets appropriately for instance, because of this problems. Then it alerts the datanode pipeline about it, by closing all the connections. 
    * When HDFS client detects a failure, it stops sending new packets to the existing pipeline. Requests and new generations stemp from a namenode, and rebuilds a pipeline from good datanodes. 
        * In this case, some packets can be resent, but there will not be extra disk IO overhead for datanodes that already saved these packets on disk. 
    * All datanodes keep track of bytes received, bytes written to a disk, and bytes acknowledged. 
    * Once the client detects a failure during close stage, it rebuilds a pipeline with good datanodes, bumps generations temp and requests to finalize replicas. 

#### Summary

* You can draw State block and replica transition tables (You should know the state transition table for a replica on a datanode and state transition table for a block on a namenode)
* You should be able to explain write pipeline behaviour, associated staged and recovery problems
* You should know four recovery processes and how they are related to each other (lease recovery → block recovery → replica recovery; pipeline recovery)

#### Questions:

* Q: Is it possible for a replica to transition from the Temporary to RUR state?
* A: No - During failures “temporary” replicas are just removed

* Q: Please specify all the possible ways to transition a replica from RWR to Finalized state
* A:
    * RWR → RUR → Finalized (It is not possible to jump between states with no connections (RWR → Finalized) but it is possible to transition via backward connections. Th right (simplified) diagram shwos the common transition workflow for a replica, whereas the left (detailed) diagram shows all the possible scenarious
    * RWR → RBW → Finalized
    * RWR → RBW → RUR → Finalized

* Q: Is it possible to transition a replica from RWR replica’s state to under_recovery?
* A: No - It is not as RWR is a replica’s state when “under_recovery” is a block’s state

### HDFS Client

* General:
    * `hdfs dfs -help`
    * `hdfs dfs -usage <utility_name>`
    * `hdfs namenode`
    * `hdfs datanode`
* List files:
    * `hdfs dfs -ls -R -h /data/wiki`
    * `hdfs dfs -du -h /data/wiki (summary of whole file system space usage)`
* Create folder:
    * `hdfs dfs -mkdir deep/nested/path --> (error)`
    * `hdfs dfs -mkdir -p deep/nested/path`
    * `hdfs dfs -ls -R deep`
* Remove folder:
    * `hdfs dfs -rm deep --> (error)`
    * `hdfs dfs -rm -r deep`
    * `hdfs dfs -mkdir -p /deep/nested/path && hdfs dfs -rm -r -skipTrash deep (deleted deep)`
* Create empty file
    * `hdfs dfs -touchz file.txt`
    * `hdfs dfs -ls`
* Move and delete file
    * `hdfs dfs -mv file.txt another_file.txt && hdfs dfs -rm another_file.txt`
* Put a local file to remote system:
    * `hdfs dfs -put <source location> <HDFS destination>`
* How to read the content of a remote file (mostly binary data)?
    * `hdfs dfs -put test_file.txt hdfs_test_file.txt`
    * `hdfs dfs -rm -skipTrash hdfs_test_file.txt (delete)`
    * `hdfs dfs -cat hdfs_test_file.txt | head -4`
    * `hdfs dfs -cat hdfs_test_file.txt | tail -4 (last 1kb of file - to reproduce locally: tail -c 1024 (byte)`
* Get remote file to local system:
    * `hdfs dfs -cp hdfs_test_file.txt hdfs_test_file_copy.txt`
    * `hdfs dfs -get hdfs_test* (download)`
    * `ls -lth hdfs*`
    * `hdfs dfs -getmerge hdfs_test* hdfs_merged.txt (merge files)`
    * `ls -lth hdfs*`
    * `cat hdfs_merged.txt (locally)`
* hdfs groups (get information about hdfs id)
    * `time hdfs dfs -setrep -w 1 hdfs_test_file.txt (Decrease or increase replication factor)`
    * `hdfs fsck /data/wiki/en_articles -files <-blocks> <-locations>`
    *  (fsck = file system checking utility; you can request name node to provide you with the information about file blocks and the allocations)`
    * `hdfs fsck -blockId blk_1073808569`
    * You can get the information about file from block id. You only need to get rid of generation stamp as it is subject to change
* Find function`
    * `hdfs dfs -find /data/wiki -name ’’*part’’`
    * `hdfs dfs -find /data/wiki -name ’’*part’’ -iname ’’*Article’’`

#### Summary:
* You can request meta-information from Namenode and chage its structure (ls, mkdir, rm, rm-r (-skipTrash), touch, mv)
* You can read and write data from and to Datanodes in HDFS (put, cat, head, tail, get, getmerge)
* You can change replication factor of files and get detailed information about data in HDFS (chown, hdfs groups; setrep; hdfs fsck; find)

#### Questions:
Q: Does “hdfs dfs -ls” count file size including the number of replicas?

A: No - this command prints the file size without replicas

### Curl
* curl is a tool to transfer data from or to a server, using one of the supported protocols (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET and TFTP)
* The command is designed to work without user interference. Thus there is no need to open a browser, type urls and download them manually.
* download one Website page
    * $ curl https://www.wikipedia.org
* download one Website page and save output to a file “wiki.html”
    * $ curl https://www.wikipedia.org -o wiki.html
* download several Website pages and save to the appropriate files
    * $ curl https://www.wikipedia.org -o wiki.html https://www.coursera.org/ -o coursera.html
* -i, --include
    * Include the HTTP response headers in the output. The HTTP response headers can include things like server name, cookies, date of the document, HTTP version and more…
    * curl -i http://www.google.com
* -L, --location
    * (HTTP) If the server reports that the requested page has moved to a different location (indicated with a Location: header and a 3XX response code), this option will make curl redo the request on the new place. If used together with -i, --include or -I, --head, headers from all requested pages will be shown.
    * curl -L http://www.google.com

### Web UI, Rest API
* Datanode information:
    * http://namenode-server:50070/dfshealth.hmtl#tab-datanode
    * Browse Directory
* HDFS Federationv
    * Huge cluster - distribute notes and meta information. With Fed there will be several name nodes - all of them independent from each other
    * They won't require any coordination between them, and will store a part of a file system entry identified by Block Pool ID, by physical ability and reliability this way.
    * Q: How is HDFS Federation scaled?
    * A: horizontally (no coordination necessary between Namnenodes, so you do not need to provide performance host machines. It is called horizontal scaling (or “scale out”)
* WebHDFS (Read-Write access)
    * Read a file content:
        * curl -i http://virtual-master.atp-fvt.org:50070/webhdfs (Request replica locations from name node) 
        * curl -i http://virtual-node1.atp-fvt.org:50070/webhdfs (Get the file data from the provided HTTP location)
        * curl -i -L http://virtual-node1.atp-fvt.org:50070/webhdfs (Get the file data from the provided HTTP location) (-L follow redirection)
    * HTTP Get (namenode)
        * Open - read data
        * Getfilestatus - get file meta information
        * Liststatus - list directory
    * HTTP Put
        * Create
        * Mkdirs
        * Rename
        * Set replication
    * HTTP Post (Transform - and add content)
        * Append
    * HTTP Delete
        * Delete

![webui_1](/Images/1_Big_Data_Essentials/Week_1/webui_1.png)

### Namenode Architecture

* Namenode is a service responsible for keeping hierarchy of folders and files
* Namenode stores all of this data in memory

Collider Example:
* 1 year of data ~ 10 PB
* Storage: 10 PB / 2 TB * 3 ~ 15k (15,000 2TB hard drives to buy) - Storage for data nodes (Replication factor of 3)
* RAM: 10 Pb / (128 Mb * 3) * 150 = 3,906,250,000 ~= 3.9Gb (because the level of granularity on the namenode is block, not replica) 
    * On average, the typical size of objects, such as folder file or block, is around 150 bytes
    * 128 Mb = default block size. When you read a block of data from a hard drive, first you need to locate on a disk (seek). Reading speed of 3.5GB/sec → 128Mb : 30-40ms. Typical drive seek time is less than one percent overhead for reading the random block of data from a hard drive and keeping block size small at the same time.
    * The more files you have in a distributed storage, the more load you have on a namenode. This load does not depend on the file size, as you have approx. the same amount of meta information stored in RAM → Small files problem
* Namenode is a single point of failure - if the service goes down, the HDFS storage became unavailable for read-only operations
    * WAL (Write ahead logging) helps you to persist changes into the storage before applying them - edit log
    * NFS (Network file system) helps you to overcome node crashes so you will be able to restore changes from a remote storage
    * Edit log is not enough to reproduce Namenode state. You should have a snapshot of memory at some point in time from which you can replay transaction stored in the edit log.

![namenode_1](/Images/1_Big_Data_Essentials/Week_1/namenode_1.png)

![namenode_2](/Images/1_Big_Data_Essentials/Week_1/namenode_2.png)

Secondary namenode or checkpoint namenode compacts the edit log by creating a new fsimage.

New fsimage is made of all the fsimage by applying all stored transaction in edit log. It is a robust and and asynchronous process.

![namenode_3](/Images/1_Big_Data_Essentials/Week_1/namenode_3.png)

#### Summary
* You can explain and reason about HDFS Namenode architecture (RAM; fsimage + edit log; block size)
* You can estimate required resources for a Hadoop cluster
* You can explain what small files problem is and where a bottleneck it
* You can list differentes between different types of Namenodes (Secondary / Checkpoint / Backup)

* Q: Please mark which of the following statements are true:
* A: Secondary Namenode == Checkpoint Namenode

### Quiz
1) What fact is more relevant to the horizontal scaling of the filesystems than to the vertical scaling?
    * A simple structure
    * **Usage of commodity hardware**
    * It provides a lower latency than the other type of scaling
2) The operation 'modify' files is not allowed in distributed FS (GFS, HDFS). What was NOT a reason to do it?
    * **Increasing reliability and accessibility**
    * Simplification of the DFS implementation
    * The data usage pattern ‘write once, read many’
3) How to achieve uniform data distribution across the servers in DFS?
    * By replication
    * By forbidding the operation 'modify' files
    * **By splitting files into blocks**
4) What does a metadata DB contain?
    * **File permissions**
    * **File creation time**
    * File content
    * **Location on the file blocks**
5) Select the correct statement about HDFS:
    * **A client requires access to all the servers to read files**
    * All the servers are equal
    * A client reads blocks of the file from a random server
6) If you have a very important file, what is the best way to protect it in HDFS?
    * Increase the replication factor for this file
    * Restrict permissions to this file
    * **Both ways are allowed and implemented in HDFS**
7) You were told that two servers in HDFS were down: Datanode and Namenode, your reaction:
    * It’s OK, replication factor is 3
    * OMG, we’ve lost everything!
    * Restore Datanode first
    * **Restore Namenode first (Is master server)**
8) What the block size in HDFS does NOT depend on?
    * Namenode RAM
    * **The block size on the local Datanodes filesystem**
    * Ratio of the block seeking time to the block reading time
