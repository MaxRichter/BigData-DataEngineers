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

![sc_core_1](/Images/1_Big_Data_Essentials/Week_4/sc_core_1.png)

#### Example: an sliced in-memory array

![sc_core_2](/Images/1_Big_Data_Essentials/Week_4/sc_core_2.png)

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

### Transformations 1

#### Two ways to construct RDDs
* Data in a stable storage (previous video)
    *Example: files in HDFS, objects in Amazon S3 bucket, lines in a text file,
    * ...
    * RDD for data in a stable storage has no dependencies
* From existing RDDs by applying a transformation (this video)
    * Example: filtered file, grouped records, ...
    * RDD for a transformed data depends on the source data
    
#### Transformations

* Allow you to create new RDDs from the existing RDDs by specifying how
to obtain new items from the existing items
* The transformed RDD depends implicitly on the source RDD

* Def: filter(p: T &rarr; Boolean): RDD[T] &rarr; RDD[T]
    * returns a filtered RDD with items satisfying the predicate p
    
![transform_1](/Images/1_Big_Data_Essentials/Week_4/transform_1.png)

* Def: map(f: T &rarr; U): RDD[T] &rarr; RDD[U]
    * returns a mapped RDD with items f(x) for every x in the source RDD

![transform_2](/Images/1_Big_Data_Essentials/Week_4/transform_2.png)    

* Def: flatMap(f: T &rarr; Array[U]): RDD[T] &rarr; RDD[U]
    * same as map but flattens the result of f
    * generalizes map and filter
    
#### Filtered RDD

* Y = X.filter(p) # where X : RDD[T]
    * Y.partitions() &rarr; Array[Partition]
        * return the same partitions as X
* Y.iterator(p: Partition, parents: Array[Iterator[T]]) &rarr; Iterator[T]
    * take a parent iterator over the corresponding partition of X
    * wrap the parent iterator to skip items that do not satisfy the
    predicate
    * return the iterator over partition of Y
* Y.dependencies() &rarr; Array[Dependency]
    * k-th partition of Y depends on k-th partition of X
    
Note that actual filtering happens not at the creation time of Y, but at hte
access time to the iterator over a partition of Y.

Same holds for other transformations - they are lazy, i.e. they compute the result
only when accessed.

#### On closures

* Y = X.filter(lambda x: x % 2 == 0)
    * predicate closure is captured within the Y (it is a part of the definition of Y)
    * predicate is not guaranteed to execute locally
    (closure may be sent over the network to the executor)

#### Partition dependency graph

Z = X.filter(lambda x: x % 2 == 0).filter(lambda y: y < 3)

![transform_3](/Images/1_Big_Data_Essentials/Week_4/transform_3.png)

#### Quiz

* Q: How can I modify in-place every item of the dataset?
* A: You cannot modify data in-place in Spark (They are immutable)
* Q: When and where is the function actually applied to the items, if you invoke the 'map' transformation on the dataset?
* A: In the future, when the data is actually requested; on the executor machine which may or may not be the local machine
* Q: Does a 'flatMap' transformation change the partitioning of the dataset?
* A: No

### Transformations 2

#### Keyed Transformations

* Def: groupByKey(): RDD[(K, V)] &rarr; RDD[(K, Array[V])]
    * groups all values with the same key into the array
    * returns a set of the arrays with corresponding keys

![transform_4](/Images/1_Big_Data_Essentials/Week_4/transform_4.png)
    
* Def: reduceByKey(f: (V, V) &rarr; V): RDD[(K, V)] &rarr; RDD[(K, V)]
    * folds all values with the same key using the given function f
    * returns a set of the folded values with corresponding keys

![transform_5](/Images/1_Big_Data_Essentials/Week_4/transform_5.png)

* Def: X.cogroup(Y: RDD[(K, W)]):RDD[(K, V)] &rarr; RDD[(K, (Array[V], Array[W]))]
    * given two keyed RDDs, groups all values with the same key
    * returns a triple (k, X-values, Y-value) for every key where X-values are all
    values found under the key k in X and Y-values are similar

![transform_6](/Images/1_Big_Data_Essentials/Week_4/transform_6.png)

How to compute an inner join from the result of cogroup?
That is, all triples (k,x,y) where (k,x) is in X and (k,y) is in Y.

#### Joins

* Def: X.join(Y: RDD[(K, W)]): RDD[(K, V)] &rarr; RDD[(K, V, W)]
    * given two keyed RDDs, returns all matching items in two datasets
    * that are triples (k, x, y) where (k, x) is in X and (k, y) is in Y
* Also: X.leftOuterJoin, X.rightOuterJoin, X.fullOuterJoin

The join transformation produces the inner join of two data sets. 
Besides that, there is the full outer join transformation. 
The difference between the inner join and the full outer join is how to treat single-sided keys. 
For the inner join, if the key is present only in the one side of the join that is in one data set. 
Then it is omitted from the result. 
For the outer join, one-sided keys are added to the result with appropriate null values.

![transform_7](/Images/1_Big_Data_Essentials/Week_4/transform_7.png)

#### Grouped RDD

* Y = X.groupByKey(): RDD[(K, V)] &rarr; RDD[(K, Array[V])]
    * Y.partitions() &rarr; Array[Partition]
        * returns a set of partitions of the key space
    * Y.iterator(p: Partition, parents: Array[Iterator[(K,V)]]) &rarr; Iterator[(K, Array[V])]												
	    * iterate over every parent partition to select pairs with the key in the
        partition range, group the pairs by the key – a shuffle operation!
        * return an iterator over the result
    * Y.dependencies() &rarr; Array[Dependency]
        * k-th output partition depends on all input partitions
        
#### Grouped RDD Shuffle

![transform_8](/Images/1_Big_Data_Essentials/Week_4/transform_8.png)

#### Narrow and Wide dependencies

![transform_9](/Images/1_Big_Data_Essentials/Week_4/transform_9.png)

#### Plenty of transformations!

![transform_10](/Images/1_Big_Data_Essentials/Week_4/transform_10.png)

#### MapReduce in Spark

![transform_11](/Images/1_Big_Data_Essentials/Week_4/transform_11.png)

#### Summary

* Transformation
    * Is a description of how to obtain a new RDD from existing RDDs
    * Is the primary way to “modify” data (given that RDDs are immutable)
* Transformations are lazy, i.e. no work is done until data is explicitly
requested (next video!)
* There are transformations with narrow and wide dependencies
* MapReduce can be expressed with a couple of transformations
* Complex transformations (like joins, cogroup) are available

#### Quiz

* Q: What is a keyed RDD?
* A: An RDD of key-value pairs
* Q: Speaking of the 'cogroup' transformation. How can you implement it with MapReduce framework?
* A: In the mapper, add an input tag (left or right) to every value in the input key-value pairs; emit the tagges values
with the original keys. In the reducer, distribute values to an array according to the value tag
* Q: Can you reproduce a cyclic dependency graph by applying Spark transformations to RDDs?
* A: No (The transformations creates a new RDD every time)

### Actions

#### Driver & executors

* Driver program runs your Spark application
* Driver delegates tasks to executors to use cluster resources
* In local mode, executors are collocated with the driver
* In cluster mode, executors are located on other machines

#### Actions

* Triggers data to be materialized and processed on the executors and
then passes the outcome to the driver
* Example: actions are used to collect, print and save data

#### Frequently used actions

* collect()
    * collects items and passes them to the driver
    * for small datasets! all data is loaded to the driver memory
* take(n: Int)
    * collects only n items and passes them to the driver
    * tries to decrease amount of computation by peeking on partitions
* top(n: Int)
    * collects n largest items and passes them to the driver
* reduce(f: (T, T) &rarr; T)
    * reduces all elements of the dataset with the given associate,
commutative binary function and passes the result back to the driver
* saveAsTextFile(path: String)
    * each executor saves its partition to a file under the given path with
every item converted to a string and confirms to the driver
* saveAsHadoopFile(path: String, outputFormatClass: String)
    * each executor saves its partition to a file under the given path using the
given Hadoop file format and confirms to the driver
* foreach(f: T &rarr; ())
    * each executor invokes f over every item and confirms to the driver
* foreachPartition(f: Iterator[T] &rarr; ())
    * each executor invokes f over its partition and confirms to the driver
    
#### Summary
* Actions trigger computation and processing of the dataset
* Actions are executed on executors and they pass results back to the
driver
* Actions are used to collect, save, print and fold data    

#### Quiz

* Q: Which file format is used to serialize data in the 'saveAsHadoopFile' action?
* A: User-provided file format, implementing Hadoop file format as in the first week

### Resiliency

#### Fault-tolerance in MapReduce

* Two key aspects
    * reliable storage for input and output data
    * deterministic and side-effect free execution of mappers and reducers

![resiliency_1](/Images/1_Big_Data_Essentials/Week_4/resiliency_1.png)
    
#### Fault-tolerance in Spark

* Same two key aspects
    * reliable storage for input and output data
    * deterministic and side-effect free execution of
    transformations(including closures)
* Determinism — every invocation of the function results in the same
    returned value
    * e. g. do not use random numbers, do not depend on a hash value order
* Freedom of side-effects — an invocation of the function does not change
anything in the external world
    * e. g. do not commit to a database, do not rely on global variables
    
#### Fault-tolerance & transformations
 * Lineage — a dependency graph for all partitions of all RDDs involved in
a computation up to the data source

![resiliency_2](/Images/1_Big_Data_Essentials/Week_4/resiliency_2.png)

![resiliency_3](/Images/1_Big_Data_Essentials/Week_4/resiliency_3.png)

![resiliency_4](/Images/1_Big_Data_Essentials/Week_4/resiliency_4.png)

#### Fault-tolerance & actions

* Actions are side-effects in Spark
* Actions have to be idempotent that is safe to be
    re-executed multiple times given the same input
* Example: collect()
    * The dataset is immutable;
      thus reading it multiple times is safe
* Example: saveAsTextFile()
    * The dataset is immutable;
    thus file would be the same after every write
    
#### Summary

* Resiliency is implemented by
    * tracking lineage
    * assuming deterministic & side-effect free execution of transformations(including closures)
    * assuming idempotency for actions
* May improve resiliency by increasing durability of RDDs in the next lesson!

#### Quiz

* Q: What is lineage?
* A: A partition dependency graph with all the partitions involved in the computation up to the data source.
* Q: What happens if the dependencies of a failed partition fails as well?
* A: Computation is restarted to recompute the dependencies first, and the partition afterwards

### Quiz

1) What functions must a dataset implement in order to be an RDD?
    * getSplits, getRecordReader
    * **partitions, iterator and dependencies**
    * foo, bar, baz
2) Mark all the things that can be used as RDDs.
    * **In-memory array**
    * Twitter firehose
    * **MySQL table** (You can partition the table by its primary key and use it as the data source)
    * **A set of CSV files in my home folder** (You can treat every file as a partition, why not?)
    * Facebook feed
    * **HDFS file**
3) Is it possible to access a MySQL database from within the predicate in the 'filter' transformation?
    * Yes, it is possible to create a database handle and capture it within the closure.
    * No, it is not possible to use database handles in predicates at all.
    * **Yes, but one need to create a database handle within the closure and close it upon returning from the predicate.**
      (However, that is not an efficient solution. A better way would be to use the 'mapPartition' transformation which 
      would allow you to reuse the handle between the predicate calls.)
4) True or false? Mark only the correct statements about the 'filter' transform.
    * **There is a single dependency on an input partition for every output partition.**
    * There may be many dependencies for some output partitions.
    * There is just one partition in the transformed RDD.
    * There are no dependencies for the output partitions.
    * **There is the same number of partitions in the transformed RDD as in the source RDD.**
    * There are indefinitely many partitions in the transformed RDD.
5) True or false? Mark only the incorrect statements.
    * **You cannot do a map-side join or a reduce-side join in Spark.**
    * **Spark natively supports only inner joins.**
    * **There is a native join transformation in Spark, and its type signature is: RDD , RDD => RDD**
    * **There is no native join transformation in Spark.**
6) Mark all the transformations with wide dependencies. Try to do this without sneaking into the documentation.
    * sample
    * flatMap
    * **join** (This transformation requires a data shuffle -- this it has wide dependencies.)
    * **cartesian** (Cartesian product is a kind of all-to-all join, it has wide dependencies.)
    * **reduceByKey** (Reduction requires data shuffling to regroup data items -- thus it has wide dependencies.)
    * **repartition**
    * filter
    * map
    * **distinct** (This is a kind of reduce-style operation, which requires a shuffle.)
7) Imagine you would like to print your dataset on the display. Which code is correct (in Python)?
    * **`myRDD.collect().map(print)`**
    * `myRDD.foreach(print)`
8) Imagine you would like to count items in your dataset. Which code is correct (in Python)?
    
    ```
    def sum_func(a, x):
        a += 1
        return a
    myRDD.fold(0, sum_func)
    # True
    ```
    
    ```
    count = 0
    myRDD.foreach(lambda x: count += 1)
    # False
    ```
    
    ```
    myRDD.reduce(lambda x, y: x + y)
    # False
    ```
9) Consider the following implementation of the 'sample' transformation:
    
    ```
    class MyRDD(RDD):
        def my_super_sample(self, ratio):
            return this.filter(lambda x: random.random() < ratio)
    ```
    Are there any issues with the implementation?
    * Yes, it is written in Python and thus very slow.
    * **Yes, it exhibits nondeterminism thus making the result non-reproducible.**
      (The major issue here is the random number generation. Two different runs over a dataset would lead to two different outcomes.)
    * No, it is completely valid implementation.
10) Consider the following action that updates a counter in a MySQL database:
    
    ```
    def update_mysql_counter():
        handle = connect_to_mysql()
        handle.execute("UPDATE counter SET value = value + 1")
        handle.close()
    myRDD.foreach(update_mysql_counter)
    ```
    Are there any issues with the implementation?
    * Yes, the action may produce incorrect results due to non-atomic increments.
    * Yes, the action is inefficient; it would be better to use 'foreachPartition'.
    * **Yes, the action may produce incorrect results due to non-idempotent updates.**
    (If the action fails while processing a partition, it would be re-executed, thus counting some items twice.)
