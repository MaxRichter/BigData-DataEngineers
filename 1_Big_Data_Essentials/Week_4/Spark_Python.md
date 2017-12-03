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


