# Introduction to Apache Spark

## Core concepts and abstractions

### Introduction

#### 2009-2012

* Key observations
    * Underutilization of cluster memory
        * for many companies data can fit into memory either now, or soon
        * memory prices were decreasing year-over-year at that time
    * Redundant disk I/O
        * especially in iterative MR jobs
    * Lack of higher-level primitives in MR
        * one has to redo joins again and again
        * one has to carefully tune the algorithm
* Key outcomes
    * RDD abstraction with rich API
    * In-memory distributed computation platform
    
#### 2012-2014

* Key observations
    * No "one system to rule them all"
        * typical cluster would include a dozen of different systems
          tailored for specific applications
        * recurrent data copying between the systems increases timings
    * Increasing demand for interactive queries and stream processing
        * due to raise of data-driven applications
            * need for fast ad-hoc analytics
            * need for fast decision-making
* Key outcomes
    * Separation of Spark Core and applications on top of the core:
    * Spark SQL
    * Spark Streaming
    * Spark GraphX
    * Spark MLlib
    
#### 2014-now

* Key observations
    * Increasing use of machine learning
    * Increasing demand for integration with other software
      (Python, R, Julia…)
* Key outcomes
    * Focus on ease-of-use
    * Spark Dataframes as first-class citizens

### RDDs

#### Why do we need new abstractions?

* Example: iterative computations (K-means, PageRank, …)
    * relation between consequent steps
    is known only to the user code , but not the framework. Therefore, the framework has no capabilities to optimize the whole computation.
    * framework must reliably persist data between steps
    (even if it is temporary data) --> thus generating excessive IO
* Example: joins
    * join operation is used in many MapReduce applications
    * not-so-easy to reuse code
    
#### Resilient Distributed Dataset

* Resilient — able to withstand failures
* Distributed — spanning across multiple machines
* Formally, a read-only, partitioned collection of records

* To adhere to RDD[T] interface, a dataset must implement:
    * partitions()  Array[Partition] (The dataset, must be able to enumerate its partitions by implementing the partition's function.)
    * iterator(p: Partition, parents: Array[Iterator[_]])  Iterator[T] (It is passed back to the iterator function of the RDD, when the framework needs to read the data from the partition.)
    * dependencies()  Array[Dependency] (The dataset must be able to enumerate its dependencies and provide an array of dependency objects.)
    * Summary: The dependency object maps partitions of the dataset to the dependencies that are partitions of the parent dataset. Those parent partitions are injected into the iterator call when creating a reader. 
* …and may implement other helper functions

* Typed! RDD[T] — a dataset of items of type T

#### Example: a (binary) data file in HDFS

![sc_core_1](/Images/1_Big_Data_Essentials/Week_2/sc_core_1.png)

#### Example: an sliced in-memory array

![sc_core_2](/Images/1_Big_Data_Essentials/Week_2/sc_core_2.png)

#### Quiz

* Q: Why is MapReduce inefficient for iterative computations?
* A: The framework has to guarantee durability of the result and hence read and write the dataset on every iteration. (Persisting datasets over and over again requires a lot of I/O time)
* Q: What does RDD stand for?
* A: Risilient Distributed Dataset
* Q: Are RDDs typed or not?
* A: Yes, every item in RDD has the same known type

#### Summary

* RDD is a read-only, partitioned collection of records
    * a developer can access the partitions and create iterators over them
    * RDD tracks dependencies (to be explained in the next video)
* Examples of RDDs
    * Hadoop files with the proper file format
    * In-memory arrays
