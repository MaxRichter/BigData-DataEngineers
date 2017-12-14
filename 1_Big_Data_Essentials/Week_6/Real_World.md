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


