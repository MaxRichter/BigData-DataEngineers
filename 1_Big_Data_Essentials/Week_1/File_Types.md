# Tuning Distributed Storage Platform with File Types

![tuning_1](/Images/1_Big_Data_Essentials/Week_1/tuning_1.png)

Data modeling:
* Data Model
    * A way you think about your data elements, what they are, what domain they come from, how different elements relate to each other, what they are composed of
    * Abstract model
    * Explicitly defines the structure of data

Relational data model:

![tuning_2](/Images/1_Big_Data_Essentials/Week_1/tuning_2.png)

![tuning_3](/Images/1_Big_Data_Essentials/Week_1/tuning_3.png)

![tuning_4](/Images/1_Big_Data_Essentials/Week_1/tuning_4.png)

* Unstructured data?
    * Technically, all data is structured at least as a byte sequence
    * Usually means “not structured for a task”
        * Ex.1: Logs = Line per request with all related data
            * Easy to work with
        * Ex. 2: Video = Sequence of frames
            * Hard to work with
* File format (also: storage format)
    * Defines (physical) data layout
    * Different design choices lead to different tradeoffs in complexity
        * affects performance, correctness
    * Primary function: to transform between raw bytes and programmatical data structures (serialization & deserialization)
    * Space efficiency
        * Different coding schemes that directly affects consumed disk space
        * Less disk space cuts your storage costs down.
    * Encoding & decoding speed
        * Space savings come at the expense of extra computation required to operate on the data → increasing CPU time
        * Poorly chosen storing format adds extra work of converting between similar representations (Store numbers in textual format)
    * Supported data types
        * Some formats can preserve type information during serialization and deserialization while others expect the user to serialize basic data types, usually strings
        * Some formats are strict and force global constraints on data while others are not (user needs to validate data and check constraints)
    * Splittable/Monolithic structure
        * Extract a subset of data without reading the entire file.
            * How to implement file splitting efficiently?
        * Think of compression or encryption as notable counterexamples
    * Extensibility
        * Think whether the existing code will break or continue working whether you add just one more extra field to your data
        * Some formats tolerate schema changes easily while others do not
* Conclusion
    * Deciding on a data model and storage format have far-reaching implications for your application performance, correctness, computation complexity and resource usage
    * Q: Why do file formats matter?
    * A: They affect application performance, correctness and extensibility

### Text formats

* CSV and TSV (comma- & tab-separated values)
    * Space efficiency
        * **BAD**
            * Not efficient for floating point or categorical data
            * ‘Ticker’: contain single repetitive value
    * Encoding/Decoding speed
        * **GOOD**
            * Simple format → very fast
    * Data types
        * **Only strings values**
            * Format gives no hint of types
            * Complexity to user code
    * Splittable
        * **Splittable w/o header**
            * Line delimited file format
    * Extensibility
        * **BAD**
            * Not easy remove or reorder columns (First field is a Ticker etc… hard coded strcuture)
* JSON (Javascript Object Notation)
    * Defines a representation for the primitive values and their combination in the form of lists and maps. 
    * Space efficiency
        * **BAD - worse than CSV (Keys are repeated)**
    * Encoding/Decoding speed
        * **Good enough**
            * JSON library for Python can parse approximately 100 to 300 Mb per second which is good enough. In C++ and Java, you can decode JSON with speed up to 1 gigabyte per second. 
    * Data types
        * Strings, Numbers, booleans, Maps, Lists
        * Eliminates serialization/deserialization issue
    * Splittable
        * Splittable if 1 document per line
    * Extensibility
        * **Yes**
            * JSON is strong in this:  You can easily add and remove fields from your data items and JSON will remain valid and parsable.
* XML (Extensible Markup Language)

Conclusion:
* Text formats are popular, human-readable, easy to generate, easy to parse
* Occupy a lot of disk space because of readability and redundancy

Questions:
* Q: In JSON, can you reliably distinguish between a floating point number and an integral number?
* A: There is no distinction between a floating point number and an integral number in the specification
* Q: How efficient is it to store floating-point values in CSV format?
* A: Rather inefficient, due to the consumed space and complex parsing

### Binary formats

Inefficiencies of text formats:
    * To parse “100500”
        * Iterate over characters: ‘1’,’0’,’0’,’5’,’0’,’0’
        * Convert them to digits: 1,0,0,5,0,0
        * Fold into the result: 1*100000 + 0*10000 + 0*1000 + 5*100 + 0*10 + 0*1
    * Not as fast as simple copying

#### Sequence Files
* First binary format implemented in Hadoop
* Stores sequence of key-value pairs
* Java-specific serialization/deserialization (not much used outside of the Java world)

![binary_1](/Images/1_Big_Data_Essentials/Week_1/binary_1.png)

* Space efficiency
    * **Moderate to Good**
        * The on-disk format closely matches the in-memory format to enable fast encoding and decoding. Strictly speaking, this statement may not hold for an arbitrary user defined code, but for primitive types, it is true.
* Encoding/Decoding speed
    * **Good**
        * Primitive values are copied as is, so there is nothing tricky
* Data types
    * **Any W/ SER./DESER. CODe**
        * Any type implement in the appropriate interfaces could be used with a format. For a developer, this is a huge advantage. You can work with custom data types, implement an arbitrary logic and use exactly the same type when interoperating with the Hadoop framework.
* Splittable
    * **Splittable**
        * Sequence files are splittable via sync markers. Sync markers are unique with a high probability, so you can seek to an arbitrary point in the file and scan for the next occurrence of the sync marker to get the next record.
* Extensibility
    * **No**
        * Not out of the box. You may include a version when serializing data and later use this version to choose among different revisions of deserialization code.

#### Avro

* Both format and support library
* Stores objects defined by schema
    * Specifies field names, types, aliases
    * Defines serialization/deserialization code
    * Allows some schema updates
* Interoperability with many languages
* What is different in Avro is that the serialization code is defined by the schema and not by the user-provided code

![binary_2](/Images/1_Big_Data_Essentials/Week_1/binary_2.png)

* Space efficiency
    * **Moderate to Good**
        * Space efficiency is similar to Sequence files. The encoding format mostly follows the in-memory format. Space savings could be obtained by using compression.
* Encoding/Decoding speed
    * **Good with Codegen**
        * Avro can generate serialization and deserialization code from a schema. In this case, its performance closely matches sequence files. Without code generation however, speed is rather limited.
* Data types
    * **JSON like**
        * Avro provides the same types as JSON, plus a few more complex types, like enumerations records. Compared to sequence files, Avro forces you to express data in the restricted type system. This is a price you pay for cross language interoperability. 
* Splittable
    * **Splittable**
        * Split ability is achieved using the same sync marker technique as in sequence files.
* Extensibility
    * **Yes**
        * Extensibility and maintainability, are design goals for Avro. So many simple operations, such as field addition, or removal, or renaming, are handled transparently by the framework. Avro is a popular format now, holding the balance between efficiency and flexibility.

##### Summary

Sequence Files and Avro are record-oriented formats. In the next video, you will learn about columnar formats. 

* Q: What are the advantages of Avro, compared to Sequence File?
* A: 
    * Better Space efficiency
    * Native support for other programming languages than Java
    * Magnitude faster encoding speed
    * Extensible data model supporting adding/removing fields
    * A fancy name and a logo
* Q: What idea is exploited in block compression in the SequenceFile format? (compared to record compression)
* A: 
    * Keys and values are better compressed in batches, because the codec has more opportunities for space savings when compressing similar items

#### Binary Formats

The execution time for analytical applications is mostly I/O bound. That means that you could gain more by optimizing input and output, while optimizing the computation has a diminishing effect on performance. 

How can you save input and output operations? 
Two options, by not reading the data necessary for the processing and by using superior compression schemes. 

![binary_3](/Images/1_Big_Data_Essentials/Week_1/binary_3.png)

First, databases were storing data row by row, linearly. They would completely serialize one row before continuing to the other. That means that, if you need to read the values from just one particular column, you still need to read the whole table. 

Columnar stores transpose data layout and store all the values column by column, enabling two key optimizations. First, you can efficiently scan only the necessary data. And second, you can achieve better compression ratios, because column-wise, data is more regular and more repetitive, and hence, more compressible.

Of course, nothing comes for free. And the price you pay when using columnar formats is the row assembly. To reconstruct the full row, you need to perform lookups from all the columns, which is likely to cause more input and output operations. However, by accessing this subset of columns, you can reduce the number of input and output operations. 

#### RCFile

Conceptually, RCFile performs horizontal, vertical data partitioning to layout data. First, rows are split into row groups. And within each row group, values are encoded column by column. 

The scheme, assuming that the row repeats with a single block managed by a single machine, ensures that the row assembly is a local operation, and hence, does not incur any network accesses. 

Every RCFile spans multiple HDFS blocks. Within every HDFS block, there is at least one row group, as defined earlier. Every row group contains three factions, sync marker, metadata, and column data. 

![binary_4](/Images/1_Big_Data_Essentials/Week_1/binary_4.png)

Metadata includes the number of rows, the number of columns, the total number of bytes, bytes per column, bytes per value. This information is used by a decoder to read the consequent column data. Metadata is compressed with the run-length encoding to save on the repeated integers. And the column data is compressed with a general-purpose codec such as ZIP. As you can see, to produce a block of data, you need to buffer a row group within the main memory and transpose it, and then precompute metadata. 

* Space efficiency
    * **Good**
        * RCFiles save a lot of this space by exploiting the columnar layout. Furthermore, data itself is compressed on the block level, increasing space savings.
* Encoding/Decoding speed
    * **Moderate to Good, Less I/O**
        * As you may notice, ZIP is not the fastest codec in the world. Speed is gained by reducing input and output operations, by not reading columns that are not used in further computation. 
* Data types
    * **Byte Strings**
        * Well, RCFiles are untyped. And values are treated as bytes. The reason for that is because RCFiles are mostly used in conjunction with Hive. And Hive deals with all the serialization and deserialization. So there is no need to offload this functionality to the format. 
* Splittable
    * **Splittable**
        * As you can see, again, sync markers are used to make a splittable format. 
* Extensibility
    * **No**
        * Encoded records have a fixed structure. So you need to deal with schema migration by yourself. Once again, this is mostly because Hive rewrites data on schema change.

#### Parquet

* The most sophisticated columnar format in Hadoop
* Collaborative effort by Twitter & Cloudera
* Supports nested and repeated data
* Exploits many columnar optimzations (such as predicate pruning, per column codecs)
* Optimizes write path

##### Summary

Binary formats are efficient in coding data:
   * SequenceFile is a reasonable choice for Java users
   * Avro is a good alternative for many use cases
   * RCFile/ORC/Parquet are best for “wide” tables and analytical workloads

Questions:
* Q: What is a row assembly?
* A: Process of reconstructing the complete row from its values from columns (price you pay when using columnar formats).
* Q: True or False. In the RCFile format, row assembly may incur network communication with other with other nodes to collect all column values.
* A: False. RCFile is horizontally-vertically partitioned. That means that data is partitioned horizontally first. In turn, this implies that every row is contained within a single block of file.

Here is a brief list of extra links that may be helpful:
* [CSV library for Python](https://docs.python.org/2/library/csv.html)
* [JSON library for Python](https://docs.python.org/2/library/json.html)
* [Apache Avro website](http://avro.apache.org/)
* [RCFile paper](http://web.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-11-4.pdf)
* [Apache Parquet website](https://parquet.apache.org/)

#### Compression

Text formats are more human friendly and widespread, thus allowing quick prototyping. You can find text logs in many applications. There are a lot of APIs that use JSON to encode data and so on. However, in text formats, there is a lot of redundancy which leads to an increased disk space usage. One common way to address space issue is to apply compression. 

* Block-level compression
    * Used in SequenceFiles, RCFiles, Parquet
    * Applied within a block of data
* File-level compression
    * Applied to the file as a whole
    * Hinders an ability to navigate through file

![compression_1](/Images/1_Big_Data_Essentials/Week_1/compression_1.png)

When to use compression:

![compression_2](/Images/1_Big_Data_Essentials/Week_1/compression_2.png)

In this example, your program spends more time computing rather than doing input and output operations. And CPU is your bottleneck. In other words, your program is CPU-bound. Adding compression would put more pressure on CPU and increase the completion time.

![compression_3](/Images/1_Big_Data_Essentials/Week_1/compression_3.png)

In this example, your program spends more time waiting for input and output, rather than doing actual computation. In other words, your program is I/O-bound. Adding compression here would allow HDFS to stream the compressed data at rate 100 MB per second, which transforms to 500 MB per second of uncompressed data, assuming the compression ratio of five. Now, your program can perform five times more work in a unit of time, which means it would complete five times faster.

* Raise awareness about application bottlenecks
    * CPU-bound → cannot benefit from the compression
    * I/O-bound → can benefit from the compression
* Codec performance vary depending on data, many options available

##### Summary

* Many applications assume relational data model
* File format defines encoding of your data
* Text formats are readable, allow quick prototyping, but are inefficient
* Binary formats are efficient, but more complicated to use
* File formats vary in terms of space efficiency, encoding & decoding speed, support for data types, extensibility
* When I/O bound, can benefit from compression
* When CPU bound, compression may increase completion time

Questions:

* Q: What does a process is CPU-bound mean?
* A: CPU is the constrained resource. Improving I/O speed will not result in any improvement of the completion time.
* Q: What does a process is I/O bound mean?
* A: The process spends more time waiting for I/O operations to complete rather than computing.

### Quiz

1) What does relational data model consist of?
    * Vertices, edges
    * Lock, stock and two smoking barrels
    * Elements, sets, relations, operators
    * **Tables, rows, columns and values**
2) Imagine that in your application you need to associate various data with users, i.e. their preferences, behavioural information and so on. You are willing to persist user profiles on the disk. Given the choice between CSV and JSON, which format would you choose?
    * CSV, obviously
    * **JSON, obviously**
    * Gatwick, obviously
3) Why are columnar file formats used in data warehousing? Mark all the correct answers.
    * **Columnar stores allow more efficient slicing of data (both horizontal and vertical).**
    * Columnar stores are highly efficient in terms of CPU usage.
    * **Columnar stores occupy less disk space due to compression**
    * Columnar stores allow efficient per-record operations like point updates.
4) Compared to text formats, why is the SequenceFile format faster? Mark all the correct statements.
    * **Simplified grammar (not tracking paired quotes, brackets, and so on) leads to a streamlined, unconditional code**
    * **Serialized data occupies less disk space thus saving I/O time**
    * **Scalar values are serialized/deserialized with a simple copy**
5) True or False? Optimizing the computation itself (optimizing the computation time) will not help to reduce the completion time of an I/O-bound process.
    * True
    * **False**
6) True or False? Switching from compressed to uncompressed data for a CPU-bound process may increase the completion time despite the saved CPU time.
    * **True**
    * False
7) What fact is more relevant to the horizontal scaling of the filesystems than to the vertical scaling?
    * A simple structure
    * It provides a lower latency than the other type of scaling
    * **Usage of commodity hardware**
8) The operation 'modify' files is not allowed in distributed FS (GFS, HDFS). What was NOT a reason to do it?
    * **Increasing reliability and accessibility**
    * The data usage pattern ‘write once, read many’
    * Simplification of the DFS implementation.
9) How to achieve uniform data distribution across the servers in DFS?
    * By forbidding the operation 'modify' files
    * By replication
    * **By splitting files into blocks**
10) What does a metadata DB contain?
    * File content
    * **File creation time**
    * **File permissions**
    * **Location on the file blocks**
11) Select the correct statement about HDFS:
    * **Namenode is a master server, it stores files metadata**
    * All the servers are equal
    * During a client writes a block it sends the block directly to all replicas
12) If you have a very important file, what is the best way to protect it in HDFS?
    * Increase the replication factor for this file
    * Restrict permissions to this file
    * **Both ways are allowed and implemented in HDFS**
13) You were told that two servers in HDFS were down: Datanode and Namenode, your reaction:
    * It’s OK, replication factor is 3
    * OMG, we’ve lost everything!
    * Restore Datanode first
    * **Restore Namenode first**
14) What the block size in HDFS does NOT depend on?
    * Ratio of the block seeking time to the block reading time
    * Namenode RAM
    * **The block size on the local Datanodes filesystem**
15) Select the correct statement about HDFS:
    * **A client requires access to all the servers to read files**
    * A client reads blocks of the file from a random server
    * All the servers are equal
