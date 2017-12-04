# Introduction to Apache Spark

## Working with Spark in Python

### Getting started with Spark & Python

#### Installing Spark locally

* Navigate to
	 *http://spark.apache.org/docs/latest/#downloading and follow the
instructions
* At the time of making this video
	* download .tar.gz
	* extract it
	* run `./bin/pyspark` from the extracted directory
* If you have IPython installed, you can run
`PYSPARK_DRIVER_PYTHON=ipython pyspark`

PySpark will load in a couple of seconds and you will be presented with a prompt as shown in the slide. 
This prompt is a regular Python interpreter with a pre initialize Spark environment. 

Remember, we were discussing the Spark context object that orchestrated all the execution in PySpark session, 
the context is created for you and you can access it with the sc variable. 

When running in the local mode the context will have the master property equals to something like local star, 
as in the slide, which means that this context is used in local executors that are threads within the same process.
When running in the cluster mode, the master URL will point to the cluster manager service like YARN or Mesos or a 
standalone SPARK cluster.

![pyspark_1](/Images/1_Big_Data_Essentials/Week_4/pyspark_1.png)

Remember the example of the in memory array there is a built in method to create an RDD from a list. 
It is called parallelize. 
Here, in the slide, I have created the RDD a from a list of numbers from 1 to 5. 
Note that when I print the a variable it says that it is a parallel collection RDD and not the list of numbers. 

This is because RDD is merely a descriptor for your data. 
I can query the number of partitions in this RDD by invoking the get numb partitions method. 
In my case it says there are four partitions. 
As we discussed, to collect the data in the trial program, I have to invoke the collect action as well as 
transformations, are just methods of an RDD object so I invoke the collect action and unsurprisingly it returns 
a list of numbers from 1 to 5. 

To complete this example let me show you how to apply transformations. 
As I said, transformations are methods of an RDD object, so I invoke the math method and pass the lambda function 
which doubles the input number. 
Again, note that the result of the transformation is an RDD. 
To compute the actually transformed dataset, I have to invoke the action once again. 

![pyspark_2](/Images/1_Big_Data_Essentials/Week_4/pyspark_2.png)

To demonstrate the laziness of RDDs, let me pull one trick. 
I'm going to do something which I told you should never do in your code. 

I'm going to create a side effect in my mapping function. 
Here, I passed the lambda function that prints the input number, doubles it, and returns the new number. 
Note, that as I am running the local mode, if my lambda function were invoked when I apply the transformation 
I would see my original numbers on the screen.

![pyspark_3](/Images/1_Big_Data_Essentials/Week_4/pyspark_3.png)

To see the numbers I have to invoke the collect action, it triggers the computation, hence invoking my lambda function, 
hence printing the numbers. 
Note that numbers are permitted because the mapping is done in parallel.

![pyspark_4](/Images/1_Big_Data_Essentials/Week_4/pyspark_4.png)

### Working with text files

#### Sample data

* Financial data from http://finance.yahoo.com for NASDAQ
* Stored in the CSV format in the file 'nasdaq.csv'

![pysparktext_1](/Images/1_Big_Data_Essentials/Week_4/pysparktext_1.png)

First, I'm going to create an RDD that wraps my text file. 
To do so, I invoke the textFile method on the Spark context object.

I'm using the take action to peek inside the RDD. 
As you can see, Spark has created the RDD where the items are the lines of the input file. 

Now, let's parse these lines. 
I'm creating a helper type just to make the code a bit more readable. 
It is easier to work with name fields rather than indexes. 

Now, I need to write a parsing code that will consume an input line and produce a record. 
My parsing function will split every line on a comma, convert all the open, high, low, close prices to floats, 
and the volume to an integer. 
Remember, in the first week, we were discussing that the CSV format is prone to parsing errors. 
Here is the perfect example. 
My code assumes the particular order of the fails in the file, so changing it will break the code. 
Yet for illustrative purposes, I think it should work just fine. 

Apply the parse record function to the dataset. 
You see, the dataset now is far more convenient to work with. 

I suggest to cache it as the pre-processing is over. This could be done by invoking the cache method. 

![pysparktext_2](/Images/1_Big_Data_Essentials/Week_4/pysparktext_2.png)

With Spark, I can easily explore the dataset. 
For example, I can compute the first and the last trading dates in my dataset. 
Or I can calculate the total trade volume. 
In these examples, the min, max, and sum methods are the actions. 

![pysparktext_3](/Images/1_Big_Data_Essentials/Week_4/pysparktext_3.png)

Now, I'm going to show you how to work with keyed datasets. 
A key value pair in Python is represented by a tuple of length two. 

I'm going to annotate every record in my sample dataset with the month and the key. 
The important part here if the net function which returns the month and the record in a single table. 

Now, I can calculate the total trade volume for every month in my dataset. 
To do so, first, I need to extract the volume out of every record by applying the month value's transformation. 

Then, for every key, I need to sum all the values by applying the reduceByKey transformation. 
Given this data, I can find the month with the largest trade volume while invoking the top action with the volume 
at the sort key. 

![pysparktext_4](/Images/1_Big_Data_Essentials/Week_4/pysparktext_4.png)

Finally, I'm going to show you how to save datasets to a reliable storage in the text format. 
When saving data in text files from pyspark, every item is serialized with the str function. 

Therefore, it is a good idea to form a data explicitly before invoking the save action. 
Let's make a CSV line for every dataset entry, and save the dataset to the out directory by invoking the saveAsTextFile 
action. 

Note that this action tells every partition to write its content independently. 
First, every partition is written to a separate file called part dash partition number to avoid write conflicts. 

In my case, I have two partitions in the output directory and the marker file indicating the success. 

![pysparktext_5](/Images/1_Big_Data_Essentials/Week_4/pysparktext_5.png)

If you are willing to create a single output file, for example, if you are sure that the output is small enough, 
you can repartition the RDD by invoking the appropriate method. 

![pysparktext_6](/Images/1_Big_Data_Essentials/Week_4/pysparktext_6.png)

#### Summary
* You have learned how to:
	* load and save text files from Pyspark
	* explore datasets
	* make keyed datasets and use keyed transformations and actions

### Joins

We use the same dataset as in the "working with text files"

When working with financial data, sometimes it is convenient to have daily returns that are ratios 
of consequent daily close prices. Let me show you, how to compute the returns with Spark.

![pysparkjoin_1](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_1.png)

First, I'm going to make a key data set, where the key is the date, and the value is the close price. 
Now, the idea is to shift the data set so that for every date, there will be the close price for the previous day. 

To perform this shift, I need to be able to compute the next date to propagate the close price. 
Here is the code, the function get_next_date parses the input date, adds one more day to the time stamp, 
and formats it back to a string.

![pysparkjoin_2](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_2.png)

Now, I'm going to make a second key data set which maps the date to the previous close price. 
I just need to associate the close price with the appropriate date, that is the next date. 
As you can see, the data is now aligned.

![pysparkjoin_3](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_3.png)

I have all the necessary bits of information to compute the daily return, stored under the same key but in the different data sets. 
So now, I'm going to join these two data sets. 

Recall, the join transformation takes two keyed RDDs and produces the inner join between two data sets. 
I can invoke the lookup action which returns the list of values for the given key. 
And voila, here are the bits in a single place.

![pysparkjoin_4](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_4.png)

Note that January third is missing in the result but is present in the source data. 
Do you understand why? 

The reason is the inner join. 
Inner join omits the keys that have either side missing. 
There is no record in the input data set with a date January second, that would produce January third as its next date.

![pysparkjoin_5](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_5.png)

Compare the inner join and the left outer join. 
The latter preserves all the keys on the left side of a join. 

![pysparkjoin_6](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_6.png)

As you may guess, there is the right outer join which preserves the keys on the right side of a join.
Finally, there is a full outer join. The result contains all the keys from both left and right sides.

![pysparkjoin_7](/Images/1_Big_Data_Essentials/Week_4/pysparkjoin_7.png)

#### Summary
* You have learned how to:
	* Compute "lagged" time series to compute daily returns
	* Join datasets with Spark
	* Differentiate between inner, left, right and full outer joins
	* Use the 'lookup' action to explore a particular key in a dataset
	
### Broadcast & Accumulator variables

Again, as in the last video, I'm going to reuse the NASDAQ sample data.

Let's start with accumulator variables. 
Recall their primary purpose is to add monitoring, profiling and debugging capabilities into the application. 
It is rather hard to come up with a reasonable use of shared variables in the local mode, so I kindly ask you to use your imagination and pay attention to the capabilities rather than the actual example. 

Imagine that I have a super regressor that tells me for the given trade volume, how large it is compared to the other days. 

And for some reason, I am concerned with the total run time of this regressor. 
For example, to decide if it's worthwhile to optimize it. 
So what I would like to do is to profile how much time my code spends in the regressor. 

To measure this time, I'm going to create an accumulator and create a wrapper function to account the elapsed time.

![pysparkakku_1](/Images/1_Big_Data_Essentials/Week_4/pysparkakku_1.png)

Now, I am going to apply the regressor to my data set and then query the accumulator, you see? 
I have invoked the collect action which triggered the computation, which invoked the timed super regressor function, 
which incremented the accumulator. 

And finally in the driver, I am reading the total accumulated value. 

Recall, transformations may be recomputed. 
So I expect the accumulator value to increase if I invoke the collect action once again.

![pysparkakku_2](/Images/1_Big_Data_Essentials/Week_4/pysparkakku_2.png)

Imagine that I need to persist the data to reliable storage and I am interested in the worst case latency of the persistence operation. 
In other words, if the storage is a database, I am interested in the maximum amount of time my code spends committing a record. 

I'm going to create a custom accumulator that aggregates values by taking their maximum. 
So aggregating all the operation latencies will result in the latency of the longest operation, the quantity I am interested in. 

I start by deriving the custom accumulator class. 
Then, I implement the zero method that returns the initial_value. 
Then, I implement the addInPlace method that performs the update. 
And finally, I instantiate the accumulator with a custom class.
 
To implement the persistence logic, I am going to use the foreachPartition action which invokes a function on every partition. 
I mock the side effect with a short slip code, time it and update the accumulator. 

![pysparkakku_3](/Images/1_Big_Data_Essentials/Week_4/pysparkakku_3.png)

Okay, let's consider the broadcast variables. 

Recall the super regressor I was using early in this video. 
As you can see, there are a few magic numbers. 
And magic numbers require tuning. 
Think of model parameters. 
We have to re-fit the model from time to time to ensure its relevance. 

Let me show you how to extract these numbers into a broadcast variable to decouple the code and the data. 
I create a broadcast variable with the parameters and replace their occurrence in the regressor code. 

![pysparkakku_4](/Images/1_Big_Data_Essentials/Week_4/pysparkakku_4.png)

#### Summary
* You have learned how to:
	* Create and use accumulator variables
	* Use a custom associative and commutative operation in an accumulator
	* Create and use broadcast variables
	* Use the 'foreachPartition' action to invoke arbitrary code on a data set

### Spark UI


For illustrative purposes, I'm going to reuse the example from the joints video. That is a computation of daily returns. 

To recap, this code loads and parses records from the nasdaq.csv file. 
Then it can view two key data sets that map the date to the close price and the close price of the previous training day. 
Then the code joins these two datasets and calculates the daily returns, okay? Okay, let's move on. 

Every Spark driver spawns a Spark UI web server unless configured otherwise. 
You can obtain its URL from the Spark context object by querying the UI web URL property. 

If you open the URL in your browser, you will see the Spark user interface for this particular application. 
As you can see, it is pretty empty. 

![pysparkui_1](/Images/1_Big_Data_Essentials/Week_4/pysparkui_1.png)

So let’s kickstart a job by finding the date with the largest daily return. 
Now refresh the web interface.

![pysparkui_2](/Images/1_Big_Data_Essentials/Week_4/pysparkui_2.png)

A new completed job has appeared and its description says, it is the top action we have invoked recently. 

In this overview, you can see that there were two stages in this job, eight tests, and the total duration of the job was two seconds. 

![pysparkui_3](/Images/1_Big_Data_Essentials/Week_4/pysparkui_3.png)

Let's click on the job. 
On the Job page, you can see a visualization of the computation graph with red boxes around the stages. 
As you can see in this example, there are two drop stages, for the joined transformation and the top action. 

Let's click on the first stage that is the join. 
Here, you can see a more detailed RDD dependency graph with a cached RDD. 

And at the bottom of the page, you can see per task statistics like input size, shuffle right size, duration, garbage collection time. 

![pysparkui_4](/Images/1_Big_Data_Essentials/Week_4/pysparkui_4.png)

Now let's click on the Storage tab in the top menu. 

Here, you can see all the RDDs with the configured persistence. 
In this example, there is just one cached RDD. 
On its detailed page, you can see that the dataset has two partitions, caching the memory in more replica. 

![pysparkui_5](/Images/1_Big_Data_Essentials/Week_4/pysparkui_5.png)

Finally, in the executor step, you can see resources available to the application. 
I'm running this example on my laptop, and I have four cores and around 400 megabytes of memory available for this shell session. 
This interface is an invaluable tool to debug performance issues of your application.

![pysparkui_6](/Images/1_Big_Data_Essentials/Week_4/pysparkui_6.png)

### Cluster Mode

* Two unresolved issues
	* How to make a standalone application
	* How to run an application on a cluster

First, let's create a standalone Spark application. 
Recall our last example that is computing the daily returns. 

To transform this code into a standalone application, we need to obtain a SparkContext object. 
To construct a SparkContext object, we have to provide it a configuration. 
That is a SparkConf object. 

Two properties that have to be configured are the application name and the master URL. 
The master URL defines the way your application would connect to a cluster. 

Here, I use the string local to make the application run in the local mode. 	

![pysparkcluster_1](/Images/1_Big_Data_Essentials/Week_4/pysparkcluster_1.png)

Other choices are standalone Spark cluster, Mesos cluster, or YARN cluster. 

Setting the string YARN in the master URL will make the application run on a reachable YARN cluster. 
Configuring a cluster from the ground up is not the subject of this video, so I am going to skip it. 
If you are interested in setting up a cluster, that's a different story, not covered in this course. 

![pysparkcluster_2](/Images/1_Big_Data_Essentials/Week_4/pysparkcluster_2.png)

Now we have a Python file with the code of our application. How to run it? 

There is a special command called spark-submit that is used to manage applications on a cluster it prints a lot of messages, like that.

`$ cat myapp.py`
... lots of Python code ...

`$ spark-submit myapp.py`
... lots of Spark messages ...

![pysparkcluster_3](/Images/1_Big_Data_Essentials/Week_4/pysparkcluster_3.png)

#### Summary
* You have learned how to:
	* Create the SparkContext object
	* Correctly set the master URL
	* Launch an application with the 'spark-submit' command
