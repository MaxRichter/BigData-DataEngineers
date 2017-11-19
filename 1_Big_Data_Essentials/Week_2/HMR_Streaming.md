# Hadoop MapReduce: How to Build Reliable System from Unreliable Components

## Hadoop MapReduce Streaming Applications in Python

### Streaming

#### MapReduce Process

![process1](/Images/1_Big_Data_Essentials/Week_2/process_1.png)

![process2](/Images/1_Big_Data_Essentials/Week_2/process_2.png)

![process3](/Images/1_Big_Data_Essentials/Week_2/process_3.png)

![process4](/Images/1_Big_Data_Essentials/Week_2/process_4.png)

In a Hadoop MapReduce application, you have a stream of input key value pairs. 
You process this data with a map function and transform this data to a list of intermediate key value pairs. 
This data is aggregated by keys during shuffle and sort phase. 
Finally, you process data providing reduce function. 

#### External programm

![process5](/Images/1_Big_Data_Essentials/Week_2/process_5.png)

If you want to plug in an external program, then it is natural to communicate via standard input and output channels. 
It would be an overkill to spend the external process for each key value pair. 
That is why Hadoop developers have decided to give you more flexibility and of course responsibility. 
You have to implement your own mappers and reducers instead of just map and reduce functions. 

The functionality of mapper includes parsing input data, processing data and outputting the data in such a way that Hadoop framework will recognize keys and tell us. 

The functionality of Reducer includes parsing input data, aggregating sorted data by keys, processing data, and finally, outputting data. 

#### Word Count

Wikipedia is usually stored in HDFS in the following format, article id, tab, and article content. In this case, line count job is equivalent to the number of articles in Wikipedia. This number changes every day because people are desperate for adding some new and very reliable information there. 

![wcprocess1](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_1.png)

Q: How do you need to change it to get only the number of lines? 
A: wc -l

#### CLI

![wcprocess2](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_2.png)

1) First you have to define the path to the STREAMING_JAR, where this jar is located depends on your cluster installation. 
2) Usually, you can use locate cli to find the path to the streaming.jar. Next, you are going to execute Yarn application. So let's call yarn, jar and path to the streaming.jar, and then provide a number of arguments. 
3) Our mapper is a bash command, wc -l. 
4) For now, you don't want to execute any reducers which is fully understandable because I'm tired of them, too. That is why you should specify this flag. 
5) Then you specify what HDFS folder or file you are going to process. In my case, the Wikipedia dump was located at this address. 
6) And finally, you specify an HDFS folder for output. 

Bear in mind that if you already have this HDFS folder, then you have to remove it beforehand, otherwise you won't be able to launch MapReduce job. Here is an example of execution error:

![wcprocess3](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_3.png)

Let us take a look at the hdfs folder internals after a successful execution. In my case, I have executed MapReduce job only against a small slice of Wikipedia. By taking a look at this folder, you can say that only two mappers were executed. By the way, it is a good practice to validate your application against a small data-set. Otherwise, you will be wasting your personal time and computational resources. 
These two files contain information about the processed articles. If you sum them up, then you get 4,100 Wikipedia articles in our sample. But let us do it in a correct MapReduce way. You have to provide reducer which aggregates the number of articles from all the markers.

![wcprocess4](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_4.png)

For each input line, I sum it up through the variable called line_count. 
And in the end I print it out. 
Please also mention that I have specified exactly one reducer. 
In that case, I have a guarantee that there is only one reducer and to have exactly one value in the output. 

![wcprocess5](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_5.png)

To make your life easier and less cumbersome, you would better rub them into a shell script. 
1) For instance, if you have the following script called reducer.shell then first, you don't need any escaping. 
2) And second, you can execute it in the following way. You execute the reducer shell script during the reduce phase.

And you also have to say that you would like the file reducer.shell to be copied from the local storage to be available from MapReduce workers. 

![wcprocess6](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_6.png)

How to write and call MapReduce shell streaming application:

![wcprocess7](/Images/1_Big_Data_Essentials/Week_2/hdf_wc_7.png)

### Streaming in Python

Python way to count lines in a distributed data set

To do this job, you have to provide mapper.py which reads data from the standard input and prints it out to the standard output:

![pystream_1](/Images/1_Big_Data_Essentials/Week_2/pystream_1.png)

If you have more than one file to distribute over the workers, then you can specify them as a comma-separated list:

![pystream_2](/Images/1_Big_Data_Essentials/Week_2/pystream_2.png)

![pystream_3](/Images/1_Big_Data_Essentials/Week_2/pystream_3.png)

As I have already mentioned, it is only natural to see an empty output by writing this streaming MapReduce job. 
You can double check it with an hdfs -text command: 

![pystream_4](/Images/1_Big_Data_Essentials/Week_2/pystream_4.png)

Mapper.py simple:

![pystream_5](/Images/1_Big_Data_Essentials/Week_2/pystream_5.png)

Mapper.py advanced:

![pystream_6](/Images/1_Big_Data_Essentials/Week_2/pystream_6.png)

Reducer.py:

![pystream_7](/Images/1_Big_Data_Essentials/Week_2/pystream_7.png)

Your first MapReduce streaming application fully on Python:

![pystream_8](/Images/1_Big_Data_Essentials/Week_2/pystream_8.png)

### WordCount in Python

How to write MapReduce word count application fully in Python.

You have to learn how to define key value pairs for the input and output streams. 
By default, the prefix of a line up to the first tab character is the key and the rest of the line is clearly the tab character will be the value. 

If there is no tab character in the line, then a key is the entire line while the value is known.

![word_count_1](/Images/1_Big_Data_Essentials/Week_2/word_count_1.png)

Let me split the text into words and count them. 
For each input line, you split it into key and value where the article ID is a key and the article content is a value. 
Then you split the content into words and finally output intermediate key value pairs. 
So, everything is splitted and shattered. 

![word_count_2](/Images/1_Big_Data_Essentials/Week_2/word_count_2.png)

Let us validate mapper against a small dataset. 
I don't want to execute any reducer so I said -numReduceTasks argument to zero. 
I have a sample of Wikipedia dataset. 
Let us make sure that you have correctly identified the key and the value. 
In the output folder, you see several map output files. 
According to the random nature, you don't know which of the mappers processed the first split of data. 

![word_count_3](/Images/1_Big_Data_Essentials/Week_2/word_count_3.png)

Then let us take a look at the both of them. 
As you can see, the first chunk of data was processed by the second mapper and there is no article ID in the output. 

![word_count_4](/Images/1_Big_Data_Essentials/Week_2/word_count_4.png)

Moving on, if you see one reducer with default implementation, which does nothing, then shuffle and sort phase will be executed and you should see the sorted output. 
Be cautious to use one reducer with big datasets. 
I use it because I know that the dataset is small. 
Okay, now let's take a look at the output. 
I think it would be better to get rid of all of the configuration characters.

![word_count_5](/Images/1_Big_Data_Essentials/Week_2/word_count_5.png)

You can easily do it with a Python regular expression module. 
I use here capital W which serves to ignore all word characters. 
If I use small W, then you would get rid of all the word characters which is not what you want to get.

![word_count_6](/Images/1_Big_Data_Essentials/Week_2/word_count_6.png)

When you call it again, you will see much better picture. 
As soon as you have mapper working correctly, let us focus on the second phase, reducer.

![word_count_7](/Images/1_Big_Data_Essentials/Week_2/word_count_7.png)

You already have data aggregated by key but how does reading the input stream in data on the reducers side look like? 
If you don't remember the level of responsibility right in streaming applications, then let me remind you that it is your task to aggregate values by keys. 

![word_count_8](/Images/1_Big_Data_Essentials/Week_2/word_count_8.png)

On reduce phase, you have sorted stream of key value pairs and it is important to mention that the stream is sorted by keys. 
Then you can iterate, line by line, and keep track of the current keys to aggregate values. 

![word_count_9](/Images/1_Big_Data_Essentials/Week_2/word_count_9.png)

First, you initialize current word to none. 
This variable will help us to keep track of current key. 
Then you parse input key value pair. 
If you see the same word, then you just increase the counter. 
Otherwise, you should output aggregate stats for the previous word and update the calendar for a new key and there is a small trick to get rid of the default key which is none. 
And the last point, when you reach the end of the input, you should not forget to output accumulated stat for the last key. 

![word_count_10](/Images/1_Big_Data_Essentials/Week_2/word_count_10.png)

If you execute the whole MapReduce job with one reducer, then you get only one file in the output. 
If you take a look at the content of this file, then you see the data sorted by keys and there is only one value for each key. 
In that case, remove numReduceTasks from the argument list. 

![word_count_11](/Images/1_Big_Data_Essentials/Week_2/word_count_11.png)

![word_count_12](/Images/1_Big_Data_Essentials/Week_2/word_count_12.png)

Then my previous job will have an arbitrary number of reducers and you will see several files in the output HDFS folder. 

![word_count_13](/Images/1_Big_Data_Essentials/Week_2/word_count_13.png)

In each file, the data is sorted by keys but the keys are not globally sorted as I shuffled between reducers. 
There is a possibility to use a special thing called TotalOrderPartitioner to sort keys between reducers. 
But I will not be able to explain it to you unless your MapReduce skills are mature.

![word_count_14](/Images/1_Big_Data_Essentials/Week_2/word_count_14.png)

### Distributed Cache

Let us consider what kind of problem where you need to filter words by vocabulary. 
It can be useful if you are looking for something specific, such as popularity of male, or female names for a child. 
Or if you are going to filter offensive words out of circulations.

![cache_1](/Images/1_Big_Data_Essentials/Week_2/cache_1.png)

At first, elaborating word count example sounds pretty straightforward. 
You take a mapper, read vocabulary in memory, and filter words by this vocabulary. 

![cache_2](/Images/1_Big_Data_Essentials/Week_2/cache_2.png)

Then during the execution you have to add this vocabulary into at least up distributed files. 
Essentially, that is all what it takes. 
But wait a bit, what does a distributed file mean? 
And how does it happen that you can read this file locally? 

Let me dive deep further about this functionality. 
This functionality is called distributed cache. 
And you already use it unintentionally in all our previous Python MapReduce examples

![cache_3](/Images/1_Big_Data_Essentials/Week_2/cache_3.png)

When you call MapReduce application, NodeManagers provide containers for execution. 
And there can be several containers on each NodeManager. 
If you provide flags minus files then each of this files will be copied once by each node before any task execution.
So each container can access this data locally via created SIM links. 
Clearly, distributed cache file should not be modified by the application while the job being executed. 

![cache_4](/Images/1_Big_Data_Essentials/Week_2/cache_4.png)

* Q: Why it is not appropriate to modify in a distributed cache from a map function?
* A: If you modify a file content in a distributed cache, then every other container on the same host machine will access the modify version of this file (they are accessed via symlinks).
Therefore, if your script execution relies on it then you can get a different output.
It breaks functional paradigm and, therefere, you will have non reproducible results.

#### Cache

There are a number of ways to distribute files. 

1) The first one is -files, that you have already seen.
2) The second one is -archives, which provides the ability to better utilize network profile transmission. 
    * In this case, all archives will be un-parked on worker nodes. 
    * So, you will be able to work with profiles from mapper or reducer's grid. T
3) The third option is to distribute files in JARs. 
    * JAR stands for Java Archive. 
    * You will not pay attention to this option at all, as this course is about Python development, not Java. 
    
![cache_5](/Images/1_Big_Data_Essentials/Week_2/cache_5.png)

Let me show you how to use archives. 
1) First, you create two files. 
    * The text files will contain female and male names.
2) Then, you can create a tar archive with the following CLI comment.

![cache_6](/Images/1_Big_Data_Essentials/Week_2/cache_6.png)

As soon as you have this tar file in place, you should be able to execute the following MapReduce application. 
The only piece is left, how you are going to use it from Python script?

![cache_7](/Images/1_Big_Data_Essentials/Week_2/cache_7.png)

There will not be a lot of changes in our script. 
You have a tar file called names.tar, and the content of this tar file will be unpacked to the folder with the same name. 
.tar in the name does not mean that you have a file. 

From this folder, you can read male and female vocabularies and use it in your Python script. 

![cache_8](/Images/1_Big_Data_Essentials/Week_2/cache_8.png)

Let us compare if you have the same picture of popular names in our Wikipedia sample. 

In Wikipedia sample, the most popular male names are James and Thomas, which are least popular in external data set. 

Female names also did not preserve the order. 
It is not good or bad, you just have different data sets. 
But you already have a tool for analytical purposes. 

![cache_9](/Images/1_Big_Data_Essentials/Week_2/cache_9.png)

* Q: What options are available to distribute file in MapReduce job?
* A: 
    * -files
    * -archives
    * -libjars
    
### Environment, Counters

Distributed cache, is not the only way to pass information to script. 
You can also get job configuration options through environment variables:
 
![env_1](/Images/1_Big_Data_Essentials/Week_2/dc_cache_1.png)

When you launch MapReduce application, the framework will assign of data to available workers. 
You can access this data from your scripts. 

For instance, if you are running a mapper, then you can access the information about the file and the slides you are working on. 
Also, you can get the information if you are running mapper or reducer. 

![env_2](/Images/1_Big_Data_Essentials/Week_2/dc_cache_2.png)

It can be important if you are running the same script on map and the reduced phase.

You can also access task id within map, or reduce phase with the following environment variables. 
The first one is used to get an absolute task id, and is usually available as a task name on job tracker UI. 

The second one, is used to access the relative order of the task within map or reduced pace. 
In this example, it would be 1 and 8. 

![env_3](/Images/1_Big_Data_Essentials/Week_2/dc_cache_3.png)

I have already told that it would be better to use HDFS client, to get the first line from HDFS file. 
But let us imagine that we write distributed in MapReduce. 
And for simplicity, we have all lines of the same size, for instance, eight characters. 
Then we can do it with the following mapper application:

![env_4](/Images/1_Big_Data_Essentials/Week_2/dc_cache_4.png)

In addition to the existing environment variables, you can provide your own. 
For example, you can write a generic mapper for word count problem that will use regular expression, provided by a user to parse words. 
You can use -D flag to provide arbitrary environment variables.

![env_5](/Images/1_Big_Data_Essentials/Week_2/dc_cache_5.png)

Here, I provided a regular expression to parse words ending with numbers. 
In the output is dfs folder, you should something like that:

![env_6](/Images/1_Big_Data_Essentials/Week_2/dc_cache_6.png)

Reading environment variables is a one-way communication between Hadoop MapReduce and your application. 

There is also a backward communication channel, between framework and your script. 
For instance, you can provide information about your task progress. 

There are two types of information:
1) Status 
2) Counters

![env_7](/Images/1_Big_Data_Essentials/Week_2/dc_cache_7.png)

You can provide an arbitrary message in status for each task execution. 
It is normally used to inform a user about the processing stage. 
For instance, startup, run and clean up. 
In this script, I have laid the task status for each processed word which sounds like an overkill, but anyway it is a good reference for usage. 

On JobTracker UI, you should be able to monitor status for each task attempt. 

You can also have a number of counters to accumulate statistics of a map and and the reduced executions. 
In word count example, it is intuitively appropriate to count the number of words. 

You can easily do it providing a counter family name, called a group. 
A counter name and the value you would like to add to the counter:

![env_8](/Images/1_Big_Data_Essentials/Week_2/dc_cache_8.png)

Again, this information is available on JobTracker UI. 
Take a look at the job counters section:

![env_9](/Images/1_Big_Data_Essentials/Week_2/dc_cache_9.png)

#### Summary
* You should be able to provide environment variables during the job execution, and view them in your streaming scripts. 
* You should be able to access job configuration options, such as map input split, so that you can write distributed MapReduce head application. 
* You should be able to report progress from your screening back to Hadoop Framework. 

### Testing

As you have already seen, streaming scripts tend to grow when you would like to support more and more functionality. 
In a word count example, it can be providing the ability for a user to define a word pattern via a regular expression, report and progress back to a MapReduce framework, filtering stop words by vocabulary and so on. 

Doing it without a proper testing is almost equal to shooting yourself in the foot. 
Let me walk you through the common testing practices while you still have two sound feet:

![test_1](/Images/1_Big_Data_Essentials/Week_2/testing_1.png)

#### Unit Testing
Unit Testing is a well known approach that can save you a number of sleepless nights. 
You can easily test function edge cases by Python testing tools. 
Tremendous amount of libraries is available for free. 
For instance, take a look at the Python Testing Tools Taxonomy on official Python wiki website. 

My personal choice is the pytest library. 
It has a good integration with debugging functionality:

![test_2](/Images/1_Big_Data_Essentials/Week_2/testing_2.png)

Let me show you how to debug a simple function with pytest.

Let us consider a simple `get_words function which transforms a line of input into a list of words.
You need to prefix test function with test. 

Pytest framework will scan the file and execute all the functions with this prefix. 
Here I provided three cases:
1) Validation of parse and non empty string
2) Validation of parse and empty string
3) Validation of raising exception if there is no input at all. 

When you call pytest from CLI and see green output, everything is good. 
Otherwise, you can provide minus minus PDB flag to drop into an interactive debugger. 
To get more information about it, I suggest you to scan through the official pytest website. 

![test_3](/Images/1_Big_Data_Essentials/Week_2/testing_3.png)

#### Integration Test

The aim of testing is to validate your scripts in a reproducible environment as close as possible to the production of one. 
Emulation of MapReduce locally is a natural choice aligned with this idea. 
Remember, our first approach to solve MapReduce word count problem, we hit the pipeline you can see in the slide, or more general the following one. 

In such a way, you can validate the whole pipeline with mapper and reducer, or independent mapper or reducer with the hand-crafted input. 
This type of testing is referred to as an integration testing, because you validate how our mappers and reducers scripts are integrated with Hadoop MapReduce streaming API. 
The previous test in practice is good and fast, but it will be working out of the box if your scripts rely on MapReduce job configuration options.

![test_4](/Images/1_Big_Data_Essentials/Week_2/testing_4.png)

#### System Testing

* Q: What CLI command can be used to find the path to Hadoop "empty" config?
* A:
    * find: Scan through the files and folders recursivly
    * locate: Suggested way to find files in Unix
    
For this purpose, Hadoop MapReduce framework provides an empty config which you can use for HDFS, MapReduce and the Yarn clients. 
Again, see they locate CLI to find a path to an empty config on your Hadoop installation. 

If you provide a Hadoop empty config, then you execute the whole MapReduce application in a standalone mode. 
In this mode an HDFS client points out to a local file system, and a node manager is working on the same node. 

In this case, your streaming scripts will be able to communicate with MapReduce framework via environment variables.

![test_5](/Images/1_Big_Data_Essentials/Week_2/testing_5.png)

You will be able to read configuration of variables and validate counters correctness. 
People can refer to this type of testing as a system testing because you execute the whole pipeline end to end. 

Validation of your streaming scripts against a sample dataset is usually the final stage before shipping your code into the production system. 
If your testing dataset is small, then you should be able to compare result with add-on solution. 

![test_6](/Images/1_Big_Data_Essentials/Week_2/testing_6.png)

#### Acceptence Testing

In addition to this, if your script contain bugs, then you will find them early without wasting your time and CPU cycles. 
This type of testing is usually referred to as an acceptance testing. 

This stage would also include validation against big datasets and measuring performance or efficiency of your solution.

![test_7](/Images/1_Big_Data_Essentials/Week_2/testing_7.png)

### Quiz

1) What do you need to define for processing data with Hadoop Streaming on the Map phase:
    * Input records reader
    * Input records format - True
    * Input record processor - True
    * Output records format - True
    * Output records writer
2) What you have to define for processing data with Hadoop Streaming on the Reduce phase:
    * Input records reader
    * Input records format - True
    * Aggregation records by key - True
    * Processor of values with the same key - True
    * Output records format - True
    * Output records writer
3) In Hadoop Streaming a mapper is run on:
    * Stream of input records - True
    * Each input record
4) In Hadoop Streaming a reducer is run on:
    * Stream of input records - True
    * Each input record
    * On records with the same key - False
5) What phase of MapReduce is this code more suitable for?
```
#!/usr/bin/env python
import sys
current_id = None
value = ''
for line in sys.stdin:
    new_id, value = line.strip().split('\t', 1)
    if new_id != current_id:
        if current_id:
            print "%s\t%s" % (current_id, value)
        current_id = new_id
if current_id:
    print "%s\t%s" % (current_id, value)
```
* Map
* Reduce - True (Yes, it leaves only one record from all the records with the same key (id), it's suitable for a reducer because it requires the sorted input records)
6) What phase of MapReduce is this code more suitable for?
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
* Map - True (Yes, it filters the input records (makes a sample) without a requirement that input records are sorted) 
* Reduce
7) What function is implemented in the following mapper:
```
#!/usr/bin/env python
import sys
for line in sys.stdin:
    key, value = line.strip().split('\t', 1)
    value = value.strip().replace('x', 'y').replace('a', 'b')
    print "%s\t%s" % (key, value)
```
* grep
* cat
* tr- True
8) What function is implemented in the following reducer:
```
#!/usr/bin/env python
import sys
current_key = None
for line in sys.stdin:
    key = line.strip()
    if key != current_key:
        if current_key:
            print current_key
        current_key = key
if current_key:
    print current_key
```
* uniq -d
* uniq - True
* sort
* wc -l
9) How can the Reduce phase in Hadoop Streaming be omitted?
* Don’t specify a ‘-reducer’ parameter
* Set the number of reducers to 0 - True
* Use a trivial reducer, i.e. cat utility
10) What is a Distributed Cache in Hadoop used for?
* To cache frequently used data on the nodes
* To deliver the required files to the nodes - True
11) You have the WordCount program for Hadoop, it outputs the result in the format:
`word count`. And now you want to count the total number of unique words in the text. What changes do you need to make?
* Add another MapReduce job with special reducer
* Use Hadoop counters from the existing job - True
12) How do you pass any parameter into your Hadoop Streaming mapper script?
* By -D option for hadoop command (-D your_param=some_value) and then get it from the environment variable 'your_param'
* By specifying parameter in the mapper command, for example: -mapper 'mapper.py --yout_param some_value'
* Both methods are possible - True
13) How do you output some debug messages for you MapReduce scripts?
* print >> sys.stderr, ‘reporter:status:Some debug’ and then find in ResourceManager web-interface
* print >> sys.stderr, ‘Some debug’ and then find it in stderr log of the task in ResourceManager web-interface
* Both methods are possible - True