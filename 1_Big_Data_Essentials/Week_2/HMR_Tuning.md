# Hadoop MapReduce: How to Build Reliable System from Unreliable Components

## Hadoop MapReduce Application Tuning: Job Configuration, Comparator, Combiner, Partitioner

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

### Comparator

### Speculative Execution / Backup Tasks

### Compression

### Quiz