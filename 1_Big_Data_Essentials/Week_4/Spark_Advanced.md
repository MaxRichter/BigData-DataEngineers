# Introduction to Apache Spark

## Advanced topics

### Execution & Scheduling

#### Spark Context

* Tells your application how to access a cluster
* Coordinates processes on the cluster to run your application

![execution_1](/Images/1_Big_Data_Essentials/Week_4/execution_1.png)

#### Jobs, stages, tasks

* Task is a unit of work to be done
* Tasks are created by a job scheduler for every job stage
* Job is spawned in response to a Spark action
* Job is divided in smaller sets of tasks called stages

![execution_2](/Images/1_Big_Data_Essentials/Week_4/execution_2.png)

* Job stage is a pipelined computation spanning between materialization
boundaries (The idea behind the job stages is to pipeline computation as much as possible, avoiding the unnecessary data materializations.)
    * Not immediately executable
* Task is a job stage bound to particular partitions
    * Immediately executable
* Materialization happens when reading, shuffling or passing data to an
action
    * narrow dependencies allow pipelining
    * wide dependencies forbid it
    
#### SparkContext – other functions

* Tracks liveness of the executors
    * required to provide fault-tolerance
* Schedules multiple concurrent jobs
    * to control the resource allocation within the application
* Performs dynamic resource allocation
    * to control the resource allocation between different applications
    
#### Summary

* The SparkContext is the core of your application
* The driver communicates directly with the executors
* Execution goes as follows:
    * Action &rarr; Job &rarr; Job Stages &rarr; Tasks
* Transformations with narrow dependencies allow pipelining

### Caching & Persistence

* RDDs are partitioned
* Execution is build around the partitions
* Block is a unit of input and output in Spark

![caching_1](/Images/1_Big_Data_Essentials/Week_4/caching_1.png)

#### Controlling persistence level
* rdd.persist(storageLevel)
    * sets RDD’s storage to persist across operations after it is computed
    for the first time
    * storageLevel is a set of flags controlling the persistence, typical
    values are:
        * DISK_ONLY – save the data to the disk,
        * MEMORY_ONLY – keep the data in the memory
        * MEMORY_AND_DISK – keep the data in the memory; when out of memory – save it to the disk
        * DISK_ONLY_2, MEMORY_ONLY_2, MEMORY_AND_DISK_2	– same as about, but make two replicas
* rdd.cache() = rdd.persist(MEMORY_ONLY)  

#### Best practices
* For interactive sessions
    * cache preprocessed data
* For batch computations
    * cache dictionaries
    *cache other datasets that are accessed multiple times
* For iterative computations
    * cache static data
* And do benchmarks!   

#### Summary
* Performance may be improved by persisting data across operations
    * in interactive sessions, iterative computations and hot datasets
* You can control the persistence of a dataset
    * whether to store in the memory or on the disk
    * how many replicas to create
    
### Broadcast Variables

Previously while learning Spark, we were concerned with polarizing the computation by partitioning data and removing any communication between the tasks. 
All communication had happened by means of the shuffle. 

In this and in the next video, you will learn about the restricted forms of the shared memory available in Spark. 

* Broadcast variable is a read-only variable that is efficiently shared
    among tasks (one to many communication)
* Distribution is done by a torrent-like protocol (extremely fast!)
* Distributed efficiently compared to captured variables

#### Example 1

Imagine you are working with 1 terabyte access log for your website. 

And you would like to resolve IP addresses to countries, to get a better understanding of your visitors. This resolution is usually performed with an aid of a special database that maps IP addresses to geographical units. Less accurate versions of this database occupy about half a gigabyte of disc space, while the most accurate one occupy up to several gigabytes. Let's assume the size of 1 gigabyte for simplicity.

One way to implement your application would be to put the database into the RDD and make it join with the log. 
However, it will require shuffling 1 terabyte of data.

A better way would be to maybe so-called map-side join. 
That is to distribute the database to every mapper and query it locally. 

Now, if you are about to start 1,000 map tasks, and you are distributing the database via the closure, that would create 100 times 1 gigabyte of outgoing traffic which is 1 terabyte at the driver load.

Distributing the database via a broadcast variable, we take slightly more than 1 gigabyte of outgoing traffic at the driver node. Maybe up to 2 gigabytes. That's 500 times to 1,000 times faster. Of course, you can replace the database in the example with the machine learning model and the argument would still hold. 

![broadcast_1](/Images/1_Big_Data_Essentials/Week_4/broadcast_1.png)

#### Example 2

Sometimes new students think that they have to put their entire computation into a single transformation graph with one action at the end.

Well, this is not true, in fact, here is what you can do with Spark. 

You can setup a transformation graph to compute a dictionary, invoke the collect action to into the driver's memory and then put it into the broadcast variable to use in further computations. 

The idea is to upload computations to spark executors and use the driver program as the coordinator. 

![broadcast_2](/Images/1_Big_Data_Essentials/Week_4/broadcast_2.png)

#### Summary
* Broadcast variables are read-only shared variables
with effective sharing mechanism
* Useful to share dictionaries, models

### Accumulator Variables

* Accumulator variable is a read-write variable that is shared among tasks
* Writes are restricted to increments!
    * i. e.: var += delta
    * addition may be replaced by any associate, commutative operation
    * Restricting the write operations allows the framework to avoid conflict synchronization, thus making the accumulators efficient.
* Reads are allowed only by the driver program and not by the executors
    * You cannot read the accumulated value from within a task.
    
#### Example

![accumulator_1](/Images/1_Big_Data_Essentials/Week_4/accumulator_1.png)

![accumulator_2](/Images/1_Big_Data_Essentials/Week_4/accumulator_2.png)

![accumulator_3](/Images/1_Big_Data_Essentials/Week_4/accumulator_3.png)

![accumulator_4](/Images/1_Big_Data_Essentials/Week_4/accumulator_4.png)

![accumulator_5](/Images/1_Big_Data_Essentials/Week_4/accumulator_5.png)

![accumulator_6](/Images/1_Big_Data_Essentials/Week_4/accumulator_6.png)

#### Guarantees on the updates
* In actions updates are applied exactly once
* In transformations there are no guarantees as the transformation code
    may be re-execute
    
#### Use cases

* Performance counters
    * number of processed records, total elapsed time, total error and so on
* Simple control flow
    * conditionals: stop on reaching a threshold for corrupted records
    * loops: decide whether to run the next iteration of an algorithm or not
* Monitoring
    * export values to the monitoring system
* Profiling & debugging

#### Summary
* Accumulators are read-write shared variables with restricted updates
    * increments only
    * can use custom associative, commutative operation for the updates
    * can read the total value only in the driver
* Useful for the control flow, monitoring, profiling & debugging

### Quiz

1) What is a job?
    * That is how Spark calls my application.
    * A pipelineable part of the computation.
    * An activity you get paid for.
    * A dependency graph for the RDDs.
    * A unit of work performed by the executor.
    * **An activity spawned in the response to a Spark action.**
2) What is a task?
    * A dependency graph for the RDDs.
    * **A unit of work performed by the executor.**
    * A pipelineable part of the computation.
    * An activity spawned in the response to a Spark action.
    * An activity you get paid for.
    * That is how Spark calls my application.
3) What is a job stage?
    * A particular shuffle operation within the job.
    * A subset of the dependency graph.
    * An activity spawned in the response to a Spark action.
    * **A pipelineable part of the computation.**
    * A single step of the job.
    * A place where a job is performed.
4) How does your application find out the executors to work with?
    * The SparkContext object queries a discovery service to find them out.
    * **The SparkContext object allocates the executors by communicating with the cluster manager.**
    * You statically define them in the configuration file.
5) Mark all the statements that are true.
    * **Data can be cached both on the disk and in the memory.** (You can tune persistence level to use both the disk & the memory)
    * **Spark can be hinted to keep particular datasets in the memory.**
    * Every partition is stored in Spark in 3 replicas to achieve fault-tolerance.
    * **You can ask Spark to make several copies of your persistent dataset.** (You can tune the replication factor)
    * Spark keeps all the intermediate data in the memory until the end of the computation, that is why it is a 'lighting-fast computing'!
    * It is advisable to cache every RDD in your computation for optimal performance.
    * While executing a job, Spark loads data from HDFS only once.
6) Imagine that you need to deliver three floating-point parameters for a machine learning algorithm used in your tasks. What is the best way to do it?
    * Hardcode them into the algorithm and redeploy the application.
    * Make a broadcast variable and put these parameters there. - False
    * **Capture them into the closure to be sent during the task scheduling.**
7) Imagine that you need to somehow print corrupted records from the log file to the screen. How can you do that?
    * Use a broadcast variable to broadcast the corrupted records and listen for these events in the driver.
    * **Use an action to collect filtered records in the driver.**
    * Use an accumulator variable to collect all the records and pass them back to the driver.
8) How broadcast variables are distributed among the executors?
    * The driver sends the content in parallel to every executor.
    * The executors are organized in a tree-like hierarchy, and the distribution follows the tree structure.
    * **The executors distribute the content with a peer-to-peer, torrent-like protocol, and the driver seeds the content.**
    * The driver sends the content one-by-one to every executor.
9) What will happen if you use a non-associative, non-commutative operator in the accumulator variables?
    * The cluster will crash.
    * I have tried that -- everything works just fine.
    * **Operation semantics are ill-defined in this case.** (As the order of the updates is unknown in advance, we must be able to apply them in any order. Thus, commutativity and associativity.)
    * Spark will not allow me to do that.
10) Mark all the operators that are both associative and commutative.
    * **sum(x, y) = x + y**
    * **prod(x, y) = x * y**
    * concat(x, y) = str(x) + str(y)
    * **min(x, y) = if x > y then y else x end**
    * avg(x, y) = (x + y) / 2
    * first(x, y) = x
    * **max(x, y) = if x > y then x else y end**
    * last(x, y) = y
11) Does Spark guarantee that accumulator updates originating from actions are applied only once?
    * **Yes.**
    * No.
12) Does Spark guarantee that accumulator updates originating from transformations are applied at least once?
    * **No.**
    * Yes.
    