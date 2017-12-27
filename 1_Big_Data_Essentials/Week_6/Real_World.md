# Real World Applications

## Working with samples

### Sampling

#### New York yellow taxi

* NYC Taxi and Limousine Commission provided anonymized data on all yellow taxi trips since 2009
* It includes information on:
    * Pick-up and drop-off times and coordinates
    * Number of passengers
    * Distance
    * Rate
    * Payment Type
    * ...
* The whole dataset is about 200 Gb

#### Preliminary analysis

* How long is a taxi trip on average?
* What is the percentage of passengers who leave tips?

`wc -l yellow_tripdata_2016-12.csv`

`10449409`

* Still too much data
* Can we take a subset?

#### Subsetting

* We can start with taking the head of the file:
`head yellow_tripdata_2016-12.csv`

![sampling_1](/Images/1_Big_Data_Essentials/Week_6/sampling_1.png)

* We could not just take head of file - trips are sorted by pick-up time

#### Random sample

* We need to shuffle the rows:
`cat yellow_tripdata_2016-12.csv | gshuf -n 10`

![sampling_2](/Images/1_Big_Data_Essentials/Week_6/sampling_2.png)

* It's shuf in ubuntu or gshuf from coreutils on Mac OS

#### Sampling taxi trips

Let's create a sample of 100 taxi trips from our file. 
First of all, make sure a new file, sample100.csv, contains a header. 

`head -n 1 yellow_tripdata_2016-12.csv > sample100.csv`

After that, we'll take all the rows except the header from the original file, shuffle them and take a sample of 100 
and then remove those weird double commas that are for some reason present at the end of each line. 

And append all of that to the sample100.csv.

`tail -n +2 yellow_tripdata_2016-12.csv | gshuf -n 100 | sed 's/''//g' >> sample100.csv`

![sampling_3](/Images/1_Big_Data_Essentials/Week_6/sampling_3.png)

### Estimating proportions

![estimations_1](/Images/1_Big_Data_Essentials/Week_6/estimations_1.png)

Let's get back to our sample of 100 taxi trips, and the question of what percentage of passengers leaves tips. 

There is a column called tip amount in the data. 
Let's create a binary vector is tipped with ones indicating whether these columns values are above zero. 

The mean of this vector and our estimate of the proportion of the tipping customers in the whole data set will be 0.66.

```
is_tipped = pd.read_csv('sample100.csv').tip_amount > 0
ph = is_tipped.mean()
ph
0.66
```
Now, is that estimate P Hat good?

#### How accurate is the estimate?

![estimations_2](/Images/1_Big_Data_Essentials/Week_6/estimations_2.png)

Can we actually quantify the accuracy of the estimate?

#### Standard Deviation

SD is a measure of the spread of the values your estimate could take on all possible samples across its mean value
for the proportion estimate P_Hat.

![estimations_3](/Images/1_Big_Data_Essentials/Week_6/estimations_3.png)

```
s = np.sqrt(ph * (1-ph) / len(is_tipped))
s
0.047
```

But is it a lot, or is it a little?
 
#### Confidence Intervall

To eliminate that vagueness we need one more concept, confidence interval. 

For the parameter P it is a pair of functions of the sample CL and CU, such that an interval from CL to CU covers P with
probability not less than 1-alpha.

![estimations_4](/Images/1_Big_Data_Essentials/Week_6/estimations_4.png)

```
from statsmodels.stats.proportion import proportion_confint

proportion_confint(sum(is_tipped), len(is_tipped), alpha=0.05)
(0.567, 0.753)
```

For our sample of 100 taxi trips, that 95% confidence interval for the proportion of tipper's could be calculated with 
the function proportional confident from the module stats models. 

It gives us the interval from 0.567 to 0.753, and it's pretty wide. 
It turns out that we are not so sure in our estimate of 66% of tipper's, with 95% confidence, the percentage of tipper's 
might be as low as 57% or as high as 75%. 

Can we get a more precise estimate? 

#### Sample Size

Sure we can. We just need a bigger sample. 

Just like point estimates get more precise with growing sample sizes, confidence intervals get narrower, but how big do 
we need N to be exactly? 

There is a special function, samplesize_confint_proportion that's giving your guess of the true proportion and the 
desired precision which is a half width of the confidence interval, returns to the required sample size.

If we want our confidence interval to be 2% wide, as you can see, we might need a sample of at least 9,108 taxi trips.

![estimations_5](/Images/1_Big_Data_Essentials/Week_6/estimations_5.png)

Let's take a sample of 10,000. In that sample we have 61% of tipper's with 95% confidence from 60.3 to 62.2. 

Indeed, the width of this interval is about 2% just like we wanted.

### Means

So we have a sample of 100 taxi trips, and we want to estimate the mean duration of the ride.

![mean_1](/Images/1_Big_Data_Essentials/Week_6/mean_1.png)

![mean_2](/Images/1_Big_Data_Essentials/Week_6/mean_2.png)

You probably can't wait to apply this to the yellow taxi sample. 

#### Average taxi trip duration

![mean_3](/Images/1_Big_Data_Essentials/Week_6/mean_3.png)

When we use the hundred times bigger sample, we obtained a 10 times narrow interval. 
It was 2 percent spike. 

We expect the same to happen with the interval for mean to duration. 
Let's see. 

![mean_4](/Images/1_Big_Data_Essentials/Week_6/mean_4.png)

So, the average duration over the 10,000 data points is 17 minutes. 
Standard deviation of the sample mean 0.61, and the final 95% confidence interval is from 16 to 18 minutes. 

Something is wrong. 
We used a hundred times more data and expected 10 times narrower interval, but its width barely changed. 

![mean_5](/Images/1_Big_Data_Essentials/Week_6/mean_5.png)

#### Histograms

To understand what happened, take a look at the histograms of the three durations over both samples. 

![mean_6](/Images/1_Big_Data_Essentials/Week_6/mean_6.png)

On the left side, it is a first sample of 100. The histogram looks nice and reasonable. 
The longest trip is about an hour and a half, which I guess might happen. 

On the right histogram, however, we could see clearly that the sample of 10,000, contains a trip that lasted about 23 hours. 
There are actually just 20 trips among those 10,000 that are above two hours. 

The main point here is that, these 20 data points of 10,000 shifted our estimate of the mean to 17 minutes. 
If we drop those observations, the sample mean will become 14 minutes. 

This is why we say that the mean is sensitive to outliers. 
If your sample contains some extreme observations, they might actually have a lot of influence on the value of the mean.

### Median

![median_1](/Images/1_Big_Data_Essentials/Week_6/median_1.png)

If you are estimating a median from a sample, you need to sort the whole sample and take the middle element. 

If the sample size is odd, it's just literally the element in the middle. 
If it is even, you should just take the average of two elements that are closest to the center. 

In a sense, median is an average value of the feature, too. 
Just like the mean, it points us to the area where the feature typically takes values. 

#### Different kind of averages

Indeed, both means and medians are called averages but they don't always coincide and some mean statisticians could use that.

![median_2](/Images/1_Big_Data_Essentials/Week_6/median_2.png)

One person earns $45,000 per year, which is probably a lot for the 50s. 
One person earns 15,000, one 10,000 and so on. 
There are 12 people whose income is $2,000. 

Now, if you need to estimate the average income of the population, you could actually calculate the sample mean, 
which will be $5,700, or the simple median, which is $3,000. 

Depending on the impression you'd like to make, you may choose one of these quantities and just report it as an average 
without specifying what kind of average it actually is. 

Most people will not notice it anyway. I just want you to be aware that averages could be manipulated.

#### Median trip duration

![median_3](/Images/1_Big_Data_Essentials/Week_6/median_3.png)

Compare that to the drastic change of the sample mean. 

To obtain a confidence interval for the median, we are going to use one of the most powerful statistical techniques 
called bootstrap. 

#### Bootstrap confidence interval

![median_4](/Images/1_Big_Data_Essentials/Week_6/median_4.png)

![median_5](/Images/1_Big_Data_Essentials/Week_6/median_5.png)

#### Median trip duration

First, we need to define two functions. 
One will generate bootstrap samples from the data, and the other will calculate the interval. 

Using those functions we calculate the interval for the median over both samples. 

A hundred data point sample gives us 95% continuous interval for the median from 8.7 to 12.4. 
The second sample of 10 thousand, from 11 to 11.4. 

It is interesting to note that with of this interval decreased about ten times as we increased the sample size 100 times, 
suggesting that the same square root of N rule applies to the bootstrap as well. 

![median_6](/Images/1_Big_Data_Essentials/Week_6/median_6.png)

Getting an interval estimate is quite important, as it helps to quantify the degree of your uncertainty in the estimate 
you provide.

### Data and Code

samples.ipynb

sample100.csv

sample10000.csv

Data Dictionary: http://www.nyc.gov/html/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf

Full dataset: http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml

### Quiz

1) Download the dataset with 10000 taxi trips below:
    sample10000.csv
    Using the data dictionary, check how many passengers in the sample paid for their ride with cash.

2) Build a 99% confidence interval for the proportion of cash payers. What is its' lower boundary?

3) Use the same sample to estimate the average trip distance in miles. Provide the answer with at least two digits after decimal.

4) What is the standard deviation of the estimator from the previous question? Provide the answer with at least three digits after decimal.

5) Calculate 95% confidence interval for the mean trip distance. What is the upper boundary? Provide the answer with at least two digits after decimal.

## Telecommunications Analytics

### Map and Reduce Side Joins


You have ten-minute intervals, aggregations of data in different locations over the city of Milan. 
The aggregated data includes the amount of SMS sent and received, the number of calls made, and the amount of Internet traffic consumed. 

You may wonder, what is the most talkative spot of the city? 
Still, we are going to validate the following hypothesis. 

Is it true that people who live in at the north of the city make twice more calls than people who live in the south? 

![join_1](/Images/1_Big_Data_Essentials/Week_6/joins_1.png)

Your data contains Square ID for each aggregated data. 
Square ID is an identificator of a polygon under the map of Milan. 

Therefore, you need to join your data with spacial data to validate the hypothesis. 
You have a big dataset of telecommunication data, and a small dataset of spacial data. 

![join_2](/Images/1_Big_Data_Essentials/Week_6/joins_2.png)

Let us solve this problem with Hadoop MapReduce.

![join_3](/Images/1_Big_Data_Essentials/Week_6/joins_3.png)

I am going to start with the simplest possible solution, and elaborate it to make it more efficient and scalable. 

When you have a big dataset, you don't have wide range of choices. 
On the mapper, you should read a piece of telecommunication data. 
Then you should somehow join this piece of data with spacial data stored in HDFS. 

No worries, you can read this data directly from your streaming script:

![join_4](/Images/1_Big_Data_Essentials/Week_6/joins_4.png)

Here on the slide, you see the Python streaming script that does this job. 
There is a special function to read the data from HDFS, and to load the Milan grid into memory. 

You call this function at the beginning of the script, and then you iterate over the lines of input with telecommunications data. 

For each line of input, you join this data by the Grid ID with spacial data. 
So you can find out if this statistics is related to the north or the south. 

![join_5](/Images/1_Big_Data_Essentials/Week_6/joins_5.png)

After running this application, you will see the following output. 
And all that happens within the map phase is of any data transferred between a mapper and a reducer. 
But what are the possible drawbacks of this solution? 

#### Distributed Cache

Usually, you have data stored with a factor of three replication. 
So, it means that you have to transfer all of this data over the network to each mapper. 
The letter usually corresponds to the number of cores on the machine. 

Do you know how to make the solution more efficient? 
Here, you have to use a distributed cache. 

This data will be replicated to each node once, and will be available locally for each member. 
Therefore, you will dramatically reduce the and increase the level of data locality.

![join_6](/Images/1_Big_Data_Essentials/Week_6/joins_6.png)

In your script, you have to change the way you access data. 
Instead of reading it from HDFS, you will read it from the local file system. 
Errors in ALT in the script is left unchanged. 

![join_7](/Images/1_Big_Data_Essentials/Week_6/joins_7.png)

When you write it again, you will see the same output. 
Please bear in mind the API of parsing an HDFS file to a distributed cache. 

You have to prefix the path with HDFS. 
And of course, you should expect the lower overall map face time. 

![join_8](/Images/1_Big_Data_Essentials/Week_6/joins_8.png)

![join_9](/Images/1_Big_Data_Essentials/Week_6/joins_9.png)

#### Map side-join

This approach which uses the distributed cached lot, small data into memory is called a Map-Side Join. 

Imagine that your telecommunication company has grown, and you have to aggregate that over thousands of cities. 
You also have a bad equipment to locate grids. 
So your spacial data is more granular and not small anymore.

![join_10](/Images/1_Big_Data_Essentials/Week_6/joins_10.png)

You need to find a way to join several big datasets
 
![join_11](/Images/1_Big_Data_Essentials/Week_6/joins_11.png)

#### Reduce side-join

As you could have guessed, if you have a Map-Side Join, then there should be a Reduce-Side Join. 

In this slide, you see two big datasets, A and B. 
During the map phase, you do nothing except parsing data into key value pairs. 
Then during the shuffle and sort phase, data is distributed by keys in a way that allows to perform the join during the reduce phase. 

The first question you may have is, how do you differentiate between two datasets in the reducer script? 

![join_12](/Images/1_Big_Data_Essentials/Week_6/joins_12.png)

You know the input block location during the map phase, so you can tag into record appropriately. 
Here, I read the environment variable, maproduce map read input file, and tag into record of spacial data by the word grid. 
Or other records, I tag by the word, logs.

![join_13](/Images/1_Big_Data_Essentials/Week_6/joins_13.png)

When you run this mapper script, you will get labels for each record into the output. 
The first column is square ID, the second is a label, and the last is a value. 

As you can see, the value has different types for different labels. 
It is a string for spacial data, and it is numeric for logs records. 

![join_14](/Images/1_Big_Data_Essentials/Week_6/joins_14.png)

Let us add the shuffle and sort phase to see how data will be distributed over the reducers:
* For some square IDs, you will see the grid records at the end. 
* For some square IDs, you will see the grid records at the beginning. 
* And for others, you can find them in the middle of logs records. 

If you would like to join the grid and log records for each square ID, then you should store in memory all the data for a specific square ID. 
It is a working solution, but there is no guarantee that you can store all the log records in memory for each square ID.

Your data can be skewed. For example, if you will not able to locate square ID for a call, then you should store null in the record. 
If you have quite a big number of records with nulls, for example, 10%, then this solution will try to log 10% of a distributed dataset in the memory on one machine. 

You do understand it is a bad idea, don't you? 

![join_15](/Images/1_Big_Data_Essentials/Week_6/joins_15.png)

You need to sort data by attack under reducer. 
This way, you can store in memory square ID, which is a location on the map. 

For example, the south or the north, and then iterate over the log records with the same square ID. 
The only problem is that in MapReduce, you can only reduce data by keys. 

You need to be careful here to make no mistakes. 
Your key is complex, and consists of two strings separated by the tab corrector, Square ID and label. 

You partition data only by square ID as in the previous examples, but you sort data by both of them.

![join_16](/Images/1_Big_Data_Essentials/Week_6/joins_16.png)

This technique is called Secondary Sort. 
Just as a reminder, if you need to sort your data in a different order, then you can use a key field based comparator available in the streaming jar. 

![join_17](/Images/1_Big_Data_Essentials/Week_6/joins_17.png)

The final command to run the Reduce-Side Join using the Secondary Sort together with the corresponding output you can see in this slide.

![join_18](/Images/1_Big_Data_Essentials/Week_6/joins_18.png)

Let me go through the reducer script to close the loop on this subject. 
You iterate over this tender output line by line. 

If you read label, then you output the previously collected statistics if there is any. 
Otherwise, you just accumulate values for the current grid. 

In your case, it is an average number of text messages received by a person in this region of a period of ten minutes.

![join_19](/Images/1_Big_Data_Essentials/Week_6/joins_19.png)

### Tabular Data

We have recently processed telecommunications data set and joint information about the received messages with Milano Grid, but there are some more factors. 
For example, the number of calls sent and received, and the amount of traffic consumed by a mobile device, which is usually not a problem till your girlfriend finds a site with kittens. 

As you have already seen, you can pass the environment variables to your stream and scripts. 
Based on these variables, you can choose which column to process. 

However, tabular data is a common for distributed file systems. 
That is why Hadoop developers have provided a special class that you can use for streaming or produce applications. 
This class is called field selection MapReduce. 

![table_1](/Images/1_Big_Data_Essentials/Week_6/table_1.png)

Field selection MapReduce has similar functionality to the CLI utility called CUT. 
You can choose which columns from a record should be considered as the key, and which columns from the record should be considered as the value.

![table_2](/Images/1_Big_Data_Essentials/Week_6/table_2.png)

For instance, in this example, I'd choose the first column, which corresponds to a square ID as a key. 
The fourth column with an index three because we enumerate from zero, and all other columns starting with index five are considered a value. 

The column sign is used to differentiate the key and value specifications. 
In this light, you can see the output of these microstrip. 

For records where you don't have enough columns, you only see partial data. 
The good thing about it is that you don't have to cover the edge cases by yourself. 

For all the other records, you see the columns starting from the first column except the column number five, which is aligned with the value specification. 

![table_3](/Images/1_Big_Data_Essentials/Week_6/table_3.png)

Let's combine several MapReduce applications into one. 
Real world applications usually consist of several steps. 
In community, it is called Job chaining.

![table_4](/Images/1_Big_Data_Essentials/Week_6/table_4.png)

#### Job Chaining

In our example, I would like to change the field selection application with the map side join. 
By the way, having small pieces of functionality is a good practice, compared to one good application that can do everything for you. 

It is much more maintainable as it would test these components independently. 
If you are writing MapReduce applications on Java or use Python packages such as Danboard, PYDOOP, HADOOPY or MrJob then these frameworks will provide that job change in functionality for your convenience. 

To clarify, I show you how you can do it by yourself. 
Having several MapReduce jobs, you should wait till the first job finishes before executing the consecutive ones. 

In this script, you can validate the return code of your application in the following way. 
In case everything is correct, the return code is zero by convention. 
Otherwise, it is different.

![table_5](/Images/1_Big_Data_Essentials/Week_6/table_5.png)

Of course, it is an oversimplification. 
Using Java and call job.waitForCompletion, the MapReduce framework calls the status of this job by the application ID every five or so seconds. 
If you'd like to mimic this behavior in bar script, then first you should find out the application ID. 

![table_6](/Images/1_Big_Data_Essentials/Week_6/table_6.png)

You can do it with yarn application - least common.
As soon as you get the application ID, you can get the status of it with the Yarn application -status.
 
![table_7](/Images/1_Big_Data_Essentials/Week_6/table_7.png)

This slide reflects the status of the running job and here, you see the status of the same job when it completes.

![table_8](/Images/1_Big_Data_Essentials/Week_6/table_8.png)

If you do not know the application ID, but you know the MapReduce application output folder, then you can check if a special file exists in this folder. 
This file is called _success. 

This empty file is used to mark at the job as successfully finished. 
This file is generated only after all the data from the MapReduce application is stored in HDFS. 

You can validate the HDFS file existence with hdfs dfs -is -e back to file, where -e means exists.

![table_9](/Images/1_Big_Data_Essentials/Week_6/table_9.png)

If you would like to prevent running in several instances, of the same application simultaneously, then you can build a synchronization mechanism via process ID. 
PID is an acronym for Process ID, and this shortcut is widely used in Unix like operating systems. 

When you spawn a process, you start the process ID in a special file, so that every other application can take a look inside and see if this process is still alive, the so-called, sure it is, cat. 
So you should put the following code at the top of your script to validate that you don't have any concurrent execution of the same script.

![table_10](/Images/1_Big_Data_Essentials/Week_6/table_10.png)

In companies with big clusters, you can have several client nodes. 
They are also called edge nodes, and are used to execute MapReduce applications. 
It means that storing the ID locally on one machine, doesn't prevent executions from other machines. 

If you can't execute your script from certain machines, then you should synchronize over unknown local storage. 
For example, you can store PID file in HDFS, with the hdfs dfs - put command. 
If this file already exists, you will get an error when you try to override it. 

To overcome the problem of still log files, you need to find a way to identify application ID. 
Occasionally, it is not easy. 
For example, see the following Stack Overflow discussion. 

Other way around, you can wrap a job with a distributed lock in service such as Oozie, Luigi Airflow, Askaban, Voldemort, wrong text. 
Who put here Harry Potter? 
Nice one there, those are fantasy fans. 
There are so many of the so-called workflow engines so you can easily find something that suits your requirements. 

Anyway, you get the idea how it works.
Let us finally count who is more talkative, northern or southern people? 

![table_11](/Images/1_Big_Data_Essentials/Week_6/table_11.png)

What is your guess? It is not difficult to write your own Python script to sum value, but there is another one Java package available for you in streaming scripts. 
This package is called aggregate. 

In the mapper output, you need to prefix each key by the type of values such as long, double, or string, and also prefix keys by action. 
For example, so mean max or unique. 

Correct and complete examples are double the value sum and stream value mean. 
Let me add this MapReduce job as the third job to our script, and reveal the secret of talkative people. 

During the examined period of time, Northern people were more talkative than people in this south. 

![table_12](/Images/1_Big_Data_Essentials/Week_6/table_12.png)

### Data Skew, Salting

How could it happen with our telecommunications data set? Is it enough? 
For example, you have a problem with our hardware or software and therefore, you are not able to locate the grid correctly. 
I hope this will never happen, but imagine that 90% of data has a null value of square ID. 

If you try to count statistic per each square ID, then all the data with null square ID will go to one machine through their reduced phase. 
Poor thing, it has to do all the job for everyone. 
It will not be feasible to process 90% of the data on one machine.
 
What should you do in this case? 
During this video, I will show you how to overcome this problem. 

The technique is called salting. 
Instead of trying to process all the data for each key on one machine, you should distribute this work of a different note. 
If you process a part of the work for a note key on one machine, and a part of the work for another machine, then you balance the load over the worker nodes. 

The solution by default will be incorrect because you have several values for the same key on different reducers. 
But you are already familiar with the concept of job chain. 

So you only need to add another one map produced stage during which you accumulate the aggregated data for different pieces of null keys. 
And this approach will become feasible because you reduced the size of data by a factor of magnitude. 

I think you have got the idea. 
Let us take a look into implementation details. 
The first question is how to distribute data with the same key into different reducers? 

You either need to change the partitioner or the target key. 
We did not cover the topic of writing your own partitioner on Java, that is why I have chosen the second option. 

![skew_1](/Images/1_Big_Data_Essentials/Week_6/skew_1.png)

Telecommunications data set doesn't have nodes. 
That is why I added this problem artificially, while doing map-side drawing. 
This slide shows you how I made 90% of square_id's node. 

Then you need to add a hash suffix for each null to distribute them over the reducer. 
Again, the Python Random model comes into play. 

![skew_2](/Images/1_Big_Data_Essentials/Week_6/skew_2.png)

When you execute our application written during the previous videos you will see the following output. 
So, the only ones step is left, you need to nourish our results back. 

The script you will do the necessary magic during the map phase to prepare the data to map produced aggregate package. 
I think you remember what stands for. 

So after adding the last number you staged, you'll see the expected output without any surfaces also called salt, that you used in the middle. 

![skew_3](/Images/1_Big_Data_Essentials/Week_6/skew_3.png)

By doing even handwaving calculations, you can approximately estimate how much time you've saved by patellizing computations. 
For example, if it takes 1000 CPU seconds to process note key on one machine, than by spreading this work over 50 independent reducers, it will take 20 CPU seconds on each of them. 

If you have these amount of available course in your head of cluster, then this job will be complete in about 20 seconds. 

You should also take into consideration extra shuffle and short phase, which will take some time. 
With the assumptions written on the screen, you'll get a wall time speed up factor around 30. 

Essentially, it is a dramatic difference when you work with big data sets.

![skew_4](/Images/1_Big_Data_Essentials/Week_6/skew_4.png)

* Q: What parameters should you take into consideration when you estimate the speedup factor of salting technique?
For example, you are trying to find out if you need to distribute the load over 10 or 100 NULL-reducers
* A: the network bandwith
* A: Shuffle & Sort time
* A: Number of available cores

Link to the dataset: 
https://dandelion.eu/datagems/SpazioDati/telecom-sms-call-internet-mi

### Quiz

1) There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make their intersection A&B?
    * Records in A: keyA, valueA
    * Records in B: keyB, valueB
    * Records in the result: key (=keyA=keyB), valueA, valueB
    
    *  **Map** (Yes, it's possible to find each keyA in B dataset in the memory on Map phase)
    * Reduce
2) There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the difference A\B (records from A not found in B)?
    * A: keyA, valueA
    * B: keyB, valueB
    * Result: keyA, valueA, null

    * **Map** (Yes, it's possible to check if each keyA exists in B dataset, B in the memory on Map phase)
    * Reduce
3) There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the difference B\A (records from B not found in A)?
    * A: keyA, valueA
    * B: keyB, valueB
    * Result: keyB, null, valueB
    
    * Map
    * **Reduce** (Yes, leave only keyB without keyA on the Reduce phase)
4) There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the union A U B (records from A or from B or from the both datasets)?
    * A: keyA, valueA
    * B: keyB, valueB
    * Result has three types of records:
        * keyA, valueA, null
        * keyB, null, valueB
        * key (=keyA=keyB), valueA, valueB

    * Map
    * **Reduce** (Yes, you can perform any joins with Reduce-side join)
5) How do you distinguish records of two datasets on the Reduce phase?
    * **By format of the values** (Yes, it's possible if the formats of two datasets are different (for example, their values contain different number of fields))
    * **By a some tag added to the records on the Map phase; tags are selected by the filename from the environment** (Yes, the filenames are known on the Map phase, use them to select a tag for each record in the mapper)
    * By the filename of dataset obtained from the environment variable
6) What parameters do you specify in the Hadoop Streaming command to perform Secondary sort (i.e. sort by two fields of the key, partition by the first field)?
    * **-D stream.num.map.output.key.fields=2** (Yes, this is required to distinguish keys from values)
    * **-D mapred.output.key.comparator.class=org.apache.hadoop.mapred.lib.KeyFieldBasedComparator** (Yes, this comparator is required to split records and sort by the fields)
    * **-D mapred.text.key.comparator.options=-k1,2** (Yes, this comparator sorts records by the first two fields)
    * **-D mapred.text.key.partitioner.options=-k1,1** (Yes, the partitioner distributes record by the first field of the key)
    * **-partitioner org.apache.hadoop.mapred.lib.KeyFieldBasedPartitioner** (Yes, this partitioner is required to calculate the partition by the specified fields of the records)
7) When is Secondary Sort really useful?
    * **When you want to avoid containers in memory on the reducers and therefore decrease the memory required by your tasks.** (Yes, Secondary Sort defines the order of input records on the reducers. So it allows to avoid using containers (trees, hash-tables) to calculate some aggregation functions (for example, 'uniq'))
    * **When you join two datasets with a Reduce-side join and one of them has many records with repeating keys** (Yes, because of Secondary Sort you know the order of the records from different datasets. It allows not to store them in memory of the reducer)
    * Always with a Reduce-side join
8) In what type of join could this code be used?
```
for line in sys.stdin:
    key, value = line.strip().split('\t', 1)
    if key in dataset:
        print "%s\t%s" % (key, value)
```    
   * **Map** (Yes, records are processed independently. There is also a search in the dataset. So, it's a mapper for a Map-side join)
   * Reduce
9) In what type of join could this script be used?
```
for line in sys.stdin:
    key, tag, value = line.strip().split('\t', 2)
    if key != current_key:
        if current_value is not None:
            print “%s\t%s” % (current_key, current_value)
        current_key = key
        current_value = None
        if tag == 0:
            continue  
    current_value = min(current_value, value) if current_value is not 
        None else value   
if current_value is not None:
    print “%s\t%s” % (current_key, current_value)
```
   * Map
   * **Reduce** (Yes, it is a reducer which removes all the records with tag=0, for other tags it implements min() function.)
10) What file is in the output directory of the succeeded MapReduce job (input the exact filename)?
    * **_SUCCESS** (Yes, a hidden (started with underscore) success file)
    
## Working with social graphs

Graphs and networks are extremely popular topics nowadays because they are everywhere. 
A social network is probably the most obvious example. 

A web-link graph, a paper citation graph, a co-purchasing network, these are other examples of large graphs that are in use today. 
Throughout this lesson, I'm going to use the Twitter graph as a data set. 
Precisely, the social graph made public by Kwak et al in 2010 as a complimentary material for their paper. 

![social_1](/Images/1_Big_Data_Essentials/Week_6/social_1.png)

#### Social Graph

What is the social graph? 
Vertexes correspond to users and edges is followed by relation. 

That is an edge from user A to user B represents and is followed by relation. 
In other words, 

![social_2](/Images/1_Big_Data_Essentials/Week_6/social_2.png)

The data set is provided in text form where every line encodes an edge in the format USER \t FOLLOWER. 
Note that the graph is not symmetric. 

If user B follows user A, user A may not follow user B. 
Here, you can see all edges incident to vertex 4825, and only the pair 2790 and 4825 is bilateral.

![social_3](/Images/1_Big_Data_Essentials/Week_6/social_3.png)

![social_4](/Images/1_Big_Data_Essentials/Week_6/social_4.png)

#### pyspark

Find the user with the largest number of followers.

![social_5](/Images/1_Big_Data_Essentials/Week_6/social_5.png)

You can combine the mapValues transformation together with the reduceByKey transformation and invoke the top action. 
Whoa, the most popular users have up to three million followers. 

Let me show you how to tidy up the code a bit. 
The only reason why you need the mapValues transformation is that you need to explicitly introduce values to aggregate for the reduceByKey transformation. 

In Spark, there is a generic transformation called aggregateByKey that allows you to use custom aggregation rules. 
It is parametrized by zero value, a value combiner that updates the aggregate given the next value, and the combiner that merges two aggregates. 

Here, I am incrementing the count by one on every value and add to aggregates upon the merge. 
Personally, I find this code cleaner with obvious intentions. 

![social_6](/Images/1_Big_Data_Essentials/Week_6/social_6.png)

One more thing, the range of follower count is extremely large. 
There are users with one follower and there are users with more than a million followers. 
This is another example of a data skew, the term you have learned from Alexey. 

If you invoke the groupBy transformation, you will run into trouble. 
The partitions will be completely skewed with the largest group requiring the most of work. 
The reason why we avoided this issue in the video is because the aggregateByKey transformation partially aggregates values on the map side, thus avoided large groups.

This is possible because the aggregate in operation is assumed to be associative and commutative. 

#### Summary
* How to load data from HDFS
* How to use the 'aggregateByKey' transform
* That social graphs exhibit extreme skewness

### Shortest path

![social_7](/Images/1_Big_Data_Essentials/Week_6/social_7.png)

Let's compute for the given user X, the distance to every other user in the graph. 
To do so, we will run a breadth-first search from the vertex X. 

Precisely, we start with annotating the vertex X with distance zero. 
Then, we find all neighbors of X and annotate them with distance one. 

On the next iteration, we start with nearly annotated vertices and repeat the process. 
When we have different distances at the vertex, we take the minimum because we are interested in the shortest path. 

We terminate the algorithm when the distances are not changing any more. 
As the distance increases with every new iteration, it is equivalent to stop when there are no unvisited vertices.

![social_8](/Images/1_Big_Data_Essentials/Week_6/social_8.png)

#### pyspark

![social_9](/Images/1_Big_Data_Essentials/Week_6/social_9.png)

The idea is to have a mapping from a vertex to its neighbors. 
Note that in this slide you have a mapping from a vertex to its followers, so you need to make a reverse mapping, let's call it forward_edges. 

To bootstrap the algorithm, let's create an initial distance mapping. 
For example, let's start with vertex 12. 

Now, you need to find neighbors of the vertex and update their distances. 
You join the distance mapping with the forward_edges, and what will be the result? 

On the left side, there is a pair of vertex 12 and distance 0. 
On the right side, there is a pair of vertex 12 and vertex 13, the neighbor vertex. 

So the joint tuple will be vertex 12, distance 0, and vertex 13. 
Now, we need to perform a step to devise that vertex 13 is at a distance of 1. 

Is it the final distance? 
Well, during the first iteration, yes, but generally speaking, no. 
Recall, you may have visited the vertex already. 

Now, how to account for that. 
You need to check if the distance was computed already or not. 

![social_10](/Images/1_Big_Data_Essentials/Week_6/social_10.png)

So, once again, you need to join two datasets as shown in this slide. 
Congratulations, you have done one iteration of the algorithm. 

Now, you need to make a loop. 
Before continuing, let's study up the code and fold these helper functions for brevity, that's it.

![social_11](/Images/1_Big_Data_Essentials/Week_6/social_11.png)

Now, let's think about the termination criteria. 
How can you check that you have visited all reachable vertices? 

For example, you can check if you have reached any new vertices on the current iteration. 
Their distance in this case will be equal to the iteration number. 

You need to create a loop, move the iteration step inside the loop and count the number of new vertices. 
Then, if there are no new vertices, you need to break the loop. 
Otherwise, prepare for the next iteration by updating the search frontier. 

If you run this code, you will notice that the iteration time becomes longer and longer.
Do you understand why?

![social_12](/Images/1_Big_Data_Essentials/Week_6/social_12.png)

There are two reasons.
First, Spark discards intermediate data after the computation. 
We can hint it to persist distances to avoid recomputing them from scratch, from the source vertex on every iteration. 

![social_13](/Images/1_Big_Data_Essentials/Week_6/social_13.png)

Second, if you look at the Spark UI, you will see that Spark reshuffles the data over and over again because it knows nothing about the key distribution of data. 
We can hint how to partition the data thus avoiding reshuffles. 
This optimization technique is covered in more detail in the next course of the specialization.

![social_14](/Images/1_Big_Data_Essentials/Week_6/social_14.png)

#### Summary
* write iterative algorithms in Spark
* tune persistence and partitioning
* implement a simple BFS graph algorithm

