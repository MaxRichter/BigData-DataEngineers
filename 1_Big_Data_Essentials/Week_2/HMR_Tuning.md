# Hadoop MapReduce: How to Build Reliable System from Unreliable Components

## Hadoop MapReduce Application Tuning: Job Configuration, Comparator, Combiner, Partitioner

The world of the efficient MapReduce is based on three whales. Combiner, Partitioner, and Comparator.
 
![intro](/Images/1_Big_Data_Essentials/Week_2/partitioner_1.png)

### Combiner

MapReduce word count application is a really good example. 
It has quite a number of parameters that you have never thought of before

![combiner1](/Images/1_Big_Data_Essentials/Week_2/combiner_1.png)

Let us consider the following line of input, word, word, then the word a and so on. 
If you use the simplest implementation, then you will get the following pairs: 

![combiner2](/Images/1_Big_Data_Essentials/Week_2/combiner_2.png)

All this data will be serialized to the local disk before and after the transmission over the network during shuffle and sort phase. 
As you can mention, there can be a lot of repetitions. 
So you would better squash the repeated items. 
For instance, from pairs word 1, word 1 we can use word 2. 
This approach can help you to dramatically change the usage of these IO operations and network bandwidth.

![combiner3](/Images/1_Big_Data_Essentials/Week_2/combiner_3.png)

You can easily do it with the following exhibit of the code. 
I hope you're familiar with the standard python collections model. 
Otherwise, you are likely to learn something new and handy.

![combiner4](/Images/1_Big_Data_Essentials/Week_2/combiner_4.png)

If you use this mapper instead of the old one, then you can see the improvement in calculations.

![combiner5](/Images/1_Big_Data_Essentials/Week_2/combiner_5.png)

But you can even be more aggressive. 
You can combine the output of several and map functions calls. 
This functionality, even has a special name in Hadoop MapReduce framework. 
It is called combiner. 
In this slide, you can see how you squash several items into one, to be more precise, a combiner has the following interface.

![combiner6](/Images/1_Big_Data_Essentials/Week_2/combiner_6.png)

It expects an input in the form of the reducer input and it has the same output signature as a mapper. 
So combiner can be applied arbitrarily number of times between map and reduce phases.

![combiner7](/Images/1_Big_Data_Essentials/Week_2/combiner_7.png)

In the word count application, there is no difference between the combiner and the reducer. 
So you can easily call it with the following arguments. 
When job finishes, you will be able to see encounters how many records were processed by the combiner. 

![combiner8](/Images/1_Big_Data_Essentials/Week_2/combiner_8.png)

In our word count example, we used the same reducer in place of the combiner. 
You're able to do it because types of key value pairs from the reducer and types of intermediate key value of pairs were the same. 
But it is not a mandatory requirement. 
Sometimes you need to write your own combiner with a different signature. 

![combiner9](/Images/1_Big_Data_Essentials/Week_2/combiner_9.png)

I'm going to show you an example, how to speed up the computation of mean values with the help of the combiner. 
Imagine you have the same Wikipedia sample, and now you're going to count how many times on average you see a word in an article. 
For simplicity, you are going to average over the number of articles contained in this or that word. 

![combiner10](/Images/1_Big_Data_Essentials/Week_2/combiner_10.png)

There are no changes in our mapper.py. 
You just count the number of words in an article and print them out:

![combiner11](/Images/1_Big_Data_Essentials/Week_2/combiner_11.png)

From the reducer's point of view, you have to memorize not only the number of occurrences but also the number of articles. 
So you will be able to average over them:

![combiner12](/Images/1_Big_Data_Essentials/Week_2/combiner_12.png)

When you try to use the combiner, then you see a dilemma. 
You cannot just average over a partial output. 
If you do this, then you lose information about how many articles we have processed. 
Therefore, the outcome of the reducer will not be correct.

![combiner13](/Images/1_Big_Data_Essentials/Week_2/combiner_13.png)

Let me change our mapper available type to a pair containing the number of articles processed and the cumulative amount of words. 
In this case, you can easily derive the mean value by dividing the cumulative amount of words, by the amount of articles.

![combiner14](/Images/1_Big_Data_Essentials/Week_2/combiner_14.png)

Here are the corresponding changes in `reducer.py`. 
Here is the code that does some spellers for each code in it, in the pair. 
It could help us to speed up calculations, for the whole MapReduce job, as you will use less IO resources. 

![combiner15](/Images/1_Big_Data_Essentials/Week_2/combiner_15.png)

Another one example, is median:

![combiner16](/Images/1_Big_Data_Essentials/Week_2/combiner_16.png)

There is a typo on the image:
* Instead of: print(current_word, word_count / article_count, sep="\t")
* Should be: print(current_word, word_count, article_count, sep="\t")

To calculate the median value precisely, you have to get the whole dataset in place. 
So, the combiner is out of help in this case. 

![combiner17](/Images/1_Big_Data_Essentials/Week_2/combiner_17.png)

* Q: Select true statements:
* A:
    * Mapper's output is Combiner's input
    * Combiner's output is Reducer's input

### Partitioner

You are already familiar with the WordCount MapReduce application and you might have got tired of it. 
Let me show you another view of this problem. 
Collocation is a sequence occur to get unusual often. 

![partitioner_2](/Images/1_Big_Data_Essentials/Week_2/partitioner_2.png)

For instance, the United States and New York are collocations. 
If you like to find collocations of size two in a data sets, for example, Wikipedia sample, then you need to count Bigrams. 

![partitioner_3](/Images/1_Big_Data_Essentials/Week_2/partitioner_3.png)

It would be the same WordCount MapReduce application. 
The only difference is that you will count bigrams of words instead of independent words, I gave to you in past time. 

![partitioner_4](/Images/1_Big_Data_Essentials/Week_2/partitioner_4.png)

The following mapper will emit a sequence of bigrams followed aggregation during their use phase. 
If you call this script is out changes, then Hadoop MapReduce framework will distribute and sort data by the first word. 

![partitioner_5](/Images/1_Big_Data_Essentials/Week_2/partitioner_5.png)

Because everything before the first type character is considered a key. 
Due to the fact that you don't have any guarantee about the way your items order, the data on a reducer will not be sorted by the second word. 

![partitioner_6](/Images/1_Big_Data_Essentials/Week_2/partitioner_6.png)

Of course, you can update `reducer.py` to count all bigrams for the first corresponding word in memory. 
Exactly as you see here, but it will be memory consuming:

```
Mapper (Python): inmemroy_bigram_reducer.py

from __future__ import print_function
from collections import Counter
import sys
current_word = None
bigram_count = Counter()
for line in sys.stdin:
    first_word, second_word, counts = line.split("\t", 2)
    counts = Counter({second_word: int(counts)})
    if first_word == current_word:
        bigram_count += counts
    else:
        if current_word:
            for second_word, bigram_count in bigram_count.items():
                print(current_word, second_word)
```

In this slide you can see the output of these MapReduce application which validates that New York bigram is a collocation. 

![partitioner_7](/Images/1_Big_Data_Essentials/Week_2/partitioner_7.png)

In addition to the unnecessary memory consumption there would be an even lot on the reducers. 
You know that there are some words which occur far more frequently than others. 
For instance, one of the most popular words in the English language, is an article, "The". 

The benefit of MapReduce that it provides functionality to particularized work. 
In a default scenario you will have the far more load on the reducer that will be busy processing this article "The". 
But you have no need to send all of the bigrams starting with "The" to one reducer as you do calculations for each pair of words independently. 

![partitioner_8](/Images/1_Big_Data_Essentials/Week_2/partitioner_8.png)

You can change the output of the mapper and substitutes the following step character with a spatial symbol which will solve your problem. 
But it would be more difficult for a user to differentiate between the volts in the diagram visually. 

As you could have already guess, here the partitioner comes into play. 
Command line arguments you can specify the way to split as a mapper or reduce your output to a key value pair. 
In this case you would like to split the line into key value pairs by the second tab character. 

This slide shows the API which helps you to do it from CLI. 
If you call it again then we should complete this MapReduce job faster due to better parallelism. 
If you list files in our output directory you should see that bigrams starting with any arbitrary word allocated in different files.

![partitioner_9](/Images/1_Big_Data_Essentials/Week_2/partitioner_9.png)

Let me show you a few more useful flags just to close the loop on the subject of data partitioning in streaming script. 

Imagine you are walking with a collection of IPv4 network addresses. 
If you have not seen them before, IPv4 address contains four numbers called Octets delimited by dots. 
The name Octet came from the fact that these numbers are limited by two in the power of eight. 

![partitioner_10](/Images/1_Big_Data_Essentials/Week_2/partitioner_10.png)

You can specify what a delimiter is and set number of fields related to a key. 
MapReduce framework will substitute this particular delimiter between num and num+1 fields to a typed character without any changes in your streaming scripts. 

In this tiny example, I told a MapReduce framework that I would like to split the output from the streaming mapper by the first dot (... stream.num.map.output.key.fields=1)

And from the reducer streaming output, I substituted the next but one dot with a key value MapReduce delimiter, which is a tab character (... stream.num.reduce.output.key.fields=2)

There are even more handy tricks that you are sure to like. 
For instance, if for some reason you'd like to partition IPv4 addresses by the second character of a first octet, then you will be able to do it with a simple CLI quote with the following arguments. 

You specify the field index and the starting character index in the start position. 
And you specify the field index and the character index in the end position (mapreduce.partition.keypartitioner.options=-k1.2,1.2).

In this case, the data will be partitioned by the letter a or b in the first octet. 
As a side note, you will never see letters in IPv4. 
I used them to highlight the point of partitioning by slides. 
And API for partition flags is equivalent for UNIX CLI sort key depth. 

As we could have mentioned, I have to set a special partitioner called KeyFieldBasedPartitioner. 
It is a Java class located in estram in Java. 
Occasionally, some partitionality is only possible to do with Java. 
You can write your own Java class to do partitioning. 
But there is already a collection of auxiliary classes that you can use to tune your stream in MapReduce application. 

From my personal experience knowing how to work with this one will be enough for the biggest amount of streaming applications that you will write. 

![partitioner_11](/Images/1_Big_Data_Essentials/Week_2/partitioner_11.png)

Moving back to a bigger picture. 
People use the following diagram to understand the whole pipeline of MapReduce application execution. 

* You have mappers at the top. 
* Then the data goes through combiners
* Then it is distributed by the partitioner. 
* Finally there is a reduced space and that is it. 

In reality functionality of combiner and partitioner is spread across the number of stages starting by in memory calculations during the map phase. 
For instance, Hadoop applies the combiner at quite a number of places. 

![partitioner_12](/Images/1_Big_Data_Essentials/Week_2/partitioner_12.png)

### Comparator

Comparator functionality is as easy as it sounds. 
All the keys in MapReduce implement writable comparable interface. 

Comparable means that you can specify the rule according to which one key is bigger than another. 
By default, you have the keys sorted by increasing order. 
For some applications, you would like to store them in a reverse order.

![comparator_1](/Images/1_Big_Data_Essentials/Week_2/comparator_1.png)

Doing some magic with a character translation, you can get the expected sort in order. 
But, Hadoop developers have already made your life easier. 

![comparator_2](/Images/1_Big_Data_Essentials/Week_2/comparator_2.png)

Take a look at the following example where I sort octets of IPV4 address by the second octet in an increasing order, and by the third octate in a reverse order. 
It is pretty much the same same interface that you have seen for Partitioner or Unix sort. 

If you would like to have numeric sort, then you should also provide flag -m.

![comparator_3](/Images/1_Big_Data_Essentials/Week_2/comparator_3.png)

* Q: If you want to sort a collection of IPV4 addresses by the second and third octect numerically and in the reverse order, then you need to provide the following arguments for KeyFieldBasedComparator:
* A: -k2,3nr

### Speculative Execution / Backup Tasks

Speculative Execution can reduce your total waiting time by a factor of two. 
If you're a Data Engineer in a company, and you manage to speed up a production data processing pipeline by a factor of two, then you can definitely ask for a promotion. 

One of the most common problems that causes a MapReduce application to wait longer for a job completion is a stroller. 
A stroller is a machine that takes an unusually long time to complete one of the last few tasks in the computation. 
It can be a result of a number of different issues:
* There can be problems with hard drive, operation system configurations, swap space uses network connections or CPU overutilization and whatnot. 

The solution for this problem was provided by the authors of MapReduce in the original article. 
They call it Backup Tasks. 

Due to the deterministic behavior of the Mapper and Reducer, you can easily re-execute straggler body of work on other note. 
In this case, the worker which processes data, they first outputs data to a distributed file system. 

All the other concurrent executions will be killed. 
Of course, the MapReduce framework is not going to have a copy for each running task. 

![spec_exec_1](/Images/1_Big_Data_Essentials/Week_2/spec_exec_1.png)

It is only used when a MapReducer application is close to completion. 
So, you usually pay less than a percent of access CPU time in return of faster job completion. 

This chart, which is taken from the original article, shows that MapReduce sort applications was faster by 30-40%. 

![spec_exec_2](/Images/1_Big_Data_Essentials/Week_2/spec_exec_2.png)

Speculative Execution is set by default to true. 
You can tune its behavior by the following properties. 

You can set these flags to false if you don't allow multiple instances of some map or reduce task to be executed in parallel. 
These two flags can be used to specify the allowed number of running backup tasks at each point in the stream of the time and overall. 

In arguments, you should specify the percentage. 
So, it would be the number between zero and one. 

![spec_exec_3](/Images/1_Big_Data_Essentials/Week_2/spec_exec_3.png)

Finally, you can tune timeouts in milliseconds that will limit the time of your waiting till the next round of speculation.

![spec_exec_4](/Images/1_Big_Data_Essentials/Week_2/spec_exec_4.png)

If you have successfully managed to speed up the process with speculation, then you should be able to find concurrent tasks healed by speculation on job trigger.

![spec_exec_5](/Images/1_Big_Data_Essentials/Week_2/spec_exec_5.png)

* Q: Speculative execution helps to overcome:
* A: Stragglers 

### Compression

You can balance the process and capacity by the data compression. 

In the first week, my colleague Yvonne provided you with the framework to analyze data compression algorithms. 
I am going to remind you the basic concepts, and to add something you've taken into consideration, data transfer in Hadoop MapReduce. 

Data compression is essentially a trade-off between the disk I/O required to read and write data. 
The network bandwidth required to send data across the network. 
And the in-memory calculation capacity, where the in-memory calculation capacity is a composite of speed and usage of CPU and RAM. 

The correct balance of these factors depends on the characteristics of your cluster, your data, your applications, or usage patterns. 

![compression_1](/Images/1_Big_Data_Essentials/Week_2/compression_1.png)

Data located in HDFS can be compressed. 
What's more, there is a shuffle and sort phase between map and the reduce where you can compress the intermediate data. 
This is exactly the place where your optimization skills can unfold.
You only need to take the following comparison table for the reference and reserve enough time for experiments. 

Splittable column means that you can cut a file at any place and find the location for the next or the previous valid record. 
For instance, it is useful to parallelize the work for mappers. 
Of course, all the compression formulas have their own pros and cons. 

* DEFLATE compression, decompression algorithm is used in DEFLATE and gzip files. 
gzip file is a deflate file with extra headers and a footer. 
* bzip is more aggressive for space requirements, but consequently, it's slower during the compression. 
The major benefit of bzip files compared to gzip ones is that they are splittable. 
* lzo files can be used in a common Hadoop scenario, where you read data far more frequently than write. 
You can provide index files for lzo files to make them splittable. 
* There is even more faster decompression algorithm called Snappy, but as you can see it has its own price. 
You will only be able to split this file records. 

As a side note, native libraries that provide implementation of compression and decompression functionality, usually also support an option to choose a trade-off between speed or space optimization. 

![compression_2](/Images/1_Big_Data_Essentials/Week_2/compression_2.png)

A Hadoop codec is an implementation of a compression, decompression algorithm. 
There are number of built-in codecs for the aforementioned compression options.

![compression_3](/Images/1_Big_Data_Essentials/Week_2/compression_3.png)

You can specify the compression parameters for intermediate data for output or for both. 
CLI arguments to tune the MapReduce application, you can see in the slide. 

![compression_4](/Images/1_Big_Data_Essentials/Week_2/compression_4.png)

Running tests is essential to see what options are the most suitable for your data processing patterns.
Here are several rules of thumb:
* gzip or bzip are a good choice for cold data, which is accessed infrequently. 
* bzip produce more compression than gzip for some kinds of files at the cost of some speed when compressing and decompressing. 
* Snappy or lzo are a better choice for hot data, which is accessed frequently. 
* Snappy often performs better than lzo. 
* For MapReduce, we can use bzip and lzo formats, if you would like to have your data splittable. 
* Snappy and gzip formats are not splittable at file level compression. 
But you can use block level compression and splittable container formats such as Avro or SequenceFile. 
In that case, we will be able to process the blocks in parallel using MapReduce. 

Henceforth, you know how to tune your MapReduce application, and what compression options are available. 

![compression_5](/Images/1_Big_Data_Essentials/Week_2/compression_5.png)

### Quiz

1) Select the facts about a combiner:
* It can significantly speed up the MapReduce job - True
* Can be implemented in any language and specified in a ‘-combiner’ option in Hadoop Streaming command - True
* It can be the same as a reducer in special cases - True
* An output format is not required to be the same as an input format
* It is run exactly once after a mapper
2) How can a partitioner be implemented and what should be specified in ‘-partitioner’ option in a Hadoop Streaming command?
* In any programming language; specify a partitioner command in a ‘-partitioner’ option
* Only in Java; specify java class in -partitions option - True
3) Select the correct statements about a partitioner:
* It is used to calculate a reducer index for each (key, value) pair - True
* Can be a non-deterministic function
* Depends on a ‘key’ field (i.e. on the field the intermediate data is sorted) or on a subset of the ‘key’ fields - True
* Can be implemented in any programming language
* Standard ‘KeyFieldBasedPartitioner’ has similar options to the Unix ‘sort’ utility - True
4) Select the correct statements about a comparator:
* Can be implemented in any programming language
* Standard ‘KeyFieldBasedComparator’ has similar options to the Unix ‘sort’ utility - True
* It can significantly speed up the MapReduce job
5) In what cases should speculative execution (of mapper, for example) be turned off?
* If the mapper is a non-deterministic function
* If the mapper has a side-effect - True ((it updates a database, requests an outer service), speculative execution should be turned off to avoid double work)
6) Select the facts about a speculative execution:
* It allows to run several instances of all the tasks of the job
* It can speed up the MapReduce job - True
* Can be the reason of a KILLED tasks status - True (tasks (or more precisely, attempts) which were successful on one node were killed on the others)
* It is turned off by default
7) Select the facts about a compression:
* Bzip2 format is splittable, i.e. one bzip2 archive can be processed by several mappers in parallel - True
* A compression can be specified both for intermediate and for output data - True
* A compression is a trade-off between CPU utilization, disk usage and ability of archives to be splitted by Hadoop - True
* A compression is a trade-off between CPU utilization and disk usage 

8.1) Select a phase in MapReduce paradigm which processes input records sorted by key:
* Map
* Shuffle & sort
* Reduce - True

8.2) What phase in MapReduce paradigm is better for filtering input records without any additional processing?
* Map
* Shuffle & sort - False
* Reduce
9) What map and reduce functions should be used (in terms of Unix utilities) to select the only unique input records?
* map=’cat’, reduce=’uniq’ - True ('uniq' on the sorted input records gives the required result)
* map=’uniq’, reduce=’uniq’ - True ( 'uniq' on the Map phase in some cases reduces the amount of records, 'uniq' on the Reduce phase gets the sorted records and solves the task)
* map=’cat’, reduce=’sort -u’
* map=’sort -u’, reduce=’sort -u’
* map=’uniq’, reduce=None
10) What map and reduce functions should be used (in terms of Unix utilities) to select only the repeated input records?
* map=’uniq’, reduce=None
* map=’uniq’, reduce=’cat’
* map=’uniq -d’, reduce=’uniq -d’
* map=’cat’, reduce=’uniq -d’ - True (mappers pass all the records to reducers and then 'uniq -d' on the sorted records solves the task)

11.1) In Hadoop Streaming a reducer is run on:
* Stream of the input records - True
* Each input record
* On records with the same key

11.2) What do you need to define for processing data with Hadoop Streaming on the Reduce phase:
* Input records reader
* Input records format - True
* Aggregation records by key - True
* Processor of values with the same key - True
* Output records format - True
* Output records writer
12) What phase of MapReduce is more suitable for this code?
```
#!/usr/bin/env python
import sys
import random
random.seed(100)
probability = float(sys.argv[1])
for line in sys.stdin:
    if random.random() <= probability:
        print line.strip()
```
* Map - True
* Reduce

13.1) How can the Reduce phase in Hadoop Streaming be omitted?
* Don't specify a '-reducer' parameter
* Set number of reducers to 0 - True
* Use a trivial reducer, i.e. cat utility

13.2) What is the Distributed Cache in Hadoop used for?
* To cache frequently used data on the nodes
* To deliver the required files to the nodes - True