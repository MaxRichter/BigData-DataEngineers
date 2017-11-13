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
