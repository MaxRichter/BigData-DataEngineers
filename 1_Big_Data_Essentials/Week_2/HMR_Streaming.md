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

### Testing

### Quiz