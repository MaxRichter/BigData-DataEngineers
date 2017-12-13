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

