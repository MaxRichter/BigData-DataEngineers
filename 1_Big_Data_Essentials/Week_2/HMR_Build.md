# Hadoop MapReduce: How to Build Reliable System from Unreliable Components

## Hadoop MapReduce: How to Build Reliable System from Unreliable Components

### Unreliable Components

When people talk about distributed systems, they often mention that such systems are built from unreliable components...

Q: 
* What do you thinj is the most popular word in Wikipedia? 
* Which word "a", "the" or "of" is more popular?
* How much more times the most popular word occurs than others?
* What is the most popular word except articles, conjunctions and other "stop" words?

A:
* The words "the", "of" and "a" occur 100 million times, 60 million times and 30 million times
* While writing Wikipedia articles, people introduce a concept once and then use it 3+ more times (100 / 3 - compare "the" vs "a")
* If you discard all the auxiliray and stop words then the most common word is "first" which you can see about 3.2 million times, the next ine is "references" with 2.9M occurrences. 

#### Node Failures

Fail-Stop

![test](/Images/1_Big_Data_Essentials/Week_2/Stop.png)
* It means that if machines get out of service during a computation then you have to have an external impact to bring system back to a working state. 
    * For instance, a system administrator should either fix the node and reboot the whole system or part of it. 
    * Or, a system administrator should retire the broken machine and reconfigure the distributed system. 
* Therefore, such distributed systems are not robust to node crashes.

Fail-Recovery

![test](/Images/1_Big_Data_Essentials/Week_2/Recovery.png)

* Fail-recovery failure means that during computations, notes can arbitrarily crash and return back to servers. 
* What is interesting, this behavior doesn't influence correctness and success of computations. 
    * That is, no external impact necessary to reconfiguring the system at such events. 
    * For instance, if a hard drive was damaged, then a system administrator can physically change the hard drive. 
* There are no other step necessary to return this node back to service. 
* After reconnection, this node will be automatically picked up by a distributed system and it will even be able to participate in current computations. 

Byzantine

![test](/Images/1_Big_Data_Essentials/Week_2/Byzantine.png)

* A distributed system is robust Byzantine failures if it can correctly work despite some of the nodes behaving out of protocol. 
* In other words, you have nodes that are going to lie through their little digital teeth to destabilize the system. 
* If you are developing a financial system, then you are likely required to deal with these types of failures to protect your customers and your business. 

#### Link Failures

Perfect Link

![test](/Images/1_Big_Data_Essentials/Week_2/perfect_link.png)

If you have a perfect link, it means that all the sent messages must be delivered and received without any modification in the same order.

Fair-Loss Link

![test](/Images/1_Big_Data_Essentials/Week_2/fail_loss_link.png)

Fair-loss link means that some part of the messages can be lost but the probability of message loss does not depend on contents of a message. Packet loss is a very common problem for network connections. For instance, the well-known TCP/IP protocol tries to solve this problem by re-transmitting messages if they were not received. 

Byzantine Link

![test](/Images/1_Big_Data_Essentials/Week_2/byzantine_link.png)

If you have byzantine links in the system, it means that some messages can be filtered according to some rule, some messages can be modified, and some messages could be created out of nowhere.

#### Distributed Booking System

![test](/Images/1_Big_Data_Essentials/Week_2/distributed_booking.png)

* Imagine you have a booking system and the only one ticket is left. You have two customers who tried to purchase a ticket more or less at the same time. 
* Then, each of the nodes of our distributed system notifies the other about tickets selling. 
* Who was there first? 
    * You can come up with an intuitive ID of adding Unix timestamp to the query time. 
    * Occasionally, it will not work correctly. 
 * Assume, you got a buyer request from user one at timestamp t_1 on node one, and you got a buyer request from users two at timestamp t_2 on node two. 
    * The node one sent message with timestamp t_1 to node two and it was received by node two at timestamp t_3. 
    * Due to clock simulation nature, it would be possible to have local t_3 time on node two less than t_1 received from node one. 
    * So, what do you have here? You have a fan fight. 
 * Who was really there first to buy the ticket in the case when timestamps are not aligns? Who cares now? Now you have to deal with the consequences.

#### Clock synchronization problem

1) Clock skew: The time can be different on different machines
2) Clock drift: There can be a different clock rate

* Any clock synchronization mechanism is subject to some precision. 
* That is why logical clocks were invented. 
    * Logical clocks help to track happened before events and therefore, order events to build reliable protocols. 
    * Logical clocks were named after his inventor, Leslie Lamport.
    
#### [A] Synchronous Systems

* Every message between nodes is delivered within limited time
* Clock drift is limited
* Each instruction execution is also limited
* At least one of this statement is always wrong in asynchronous systems

Fail-stop, perfect link, and synchronous model:
* parallel computational model

Fail-recovery, fair-loss link, and asynchronous model:
* Focus of this course

Byzantine-failure, byzantine link, and asynchronous model:
* Grid Computing (Is a model adopted for the systems where computational components spread across the globe of unreliable and untrusted network connections)

* Q: In this course you are going to focus on Hadoop ecosystem which has the following model:
* A: Fail-recovery node, fair-loss link, asynchronous model:
* Q: What is the best model? (Select from the three above)
* A: Neither of them - there is no such thing as the best model

### Map Reduce

Map 

![map](/Images/1_Big_Data_Essentials/Week_2/map.png)

Reduce

![reduce](/Images/1_Big_Data_Essentials/Week_2/reduce.png)

* Reduce function computes the value from left to right. 

Pitfall

![reduce](/Images/1_Big_Data_Essentials/Week_2/pitfall.png)

* Mean function is not associative
* So if you apply this to a list of numbers from 1 to 3, then you will get the outcome 2.25. But if you apply the same function from right to left, then you will get 1.75. 
* Thus, changing the order of the atoms does not change the sum. But changing the application order definitely effects the result. 

MapReduce

![mapreduce](/Images/1_Big_Data_Essentials/Week_2/mapreduce.png)

### Distributed Shell

Grep

![grep1](/Images/1_Big_Data_Essentials/Week_2/grep_1.png)

![grep2](/Images/1_Big_Data_Essentials/Week_2/grep_2.png)

Head

![head1](/Images/1_Big_Data_Essentials/Week_2/head_1.png)

![head2](/Images/1_Big_Data_Essentials/Week_2/head_2.png)

Word Count (WC)

![wc1](/Images/1_Big_Data_Essentials/Week_2/wc_1.png)

![wc2](/Images/1_Big_Data_Essentials/Week_2/wc_2.png)

`distributed grep: (map=grep) + (reduce=None)`

`distributed head: (map=head*) + (reduce=None)`

`distributed wc: (map=wc) + (reduce=operator.add*)`

#### World Count

![world1](/Images/1_Big_Data_Essentials/Week_2/world_1.png)

![world2](/Images/1_Big_Data_Essentials/Week_2/world_2.png)

For a distributed MapReduce application, cat is used to read data. tr is a map function to split text into words, and uniq is obviously a reduced function. Then what should you do with sort? Is it a map or a reduce?

![world3](/Images/1_Big_Data_Essentials/Week_2/world_3.png)

During the map phase, the text is split in two words. Then, during shuffle and sort phase, words are distributed to a reduce phase in a way that reduce functions can be executed independently on different machines. For word count application, it means that words are distributed by a word hash.

In a simple example, if you have 26 independent reducers, and only English words, then you could spread their words alphabetically by their first character from a to z. As you can notice, there's also a sorts tab. Even if you have all the words distributed alphabetically, you may not have enough RAM space to fit this data in memory. But if the data is sorted, and can be read as a stream, then uniq minus c will be working correctly. To make data sorted, you only need to have enough disk space. The algorithm for this is called, external sorting.

Map Reduce Formal Model:
* map: (key, value) --> (key, value)
* reduce: (key, value) --> (key, value)

![world4](/Images/1_Big_Data_Essentials/Week_2/world_4.png)

First, you read data, and get pairs with a line number, and line contact. Then, on a map phase, you ignore line numbers and split lines into words. To satisfy this model, you can add value one to each output it worked. So, it means that you have seen this word once by reading a line from left to right. Then, a shuffle and sort phase where you spread the words by the hashes. So, you can process them on independent reducers. At last, you count how many figures of one you have for each word, and thumbs them up to get an answer.
 
![world5](/Images/1_Big_Data_Essentials/Week_2/world_5.png)

![world6](/Images/1_Big_Data_Essentials/Week_2/world_6.png)

![world7](/Images/1_Big_Data_Essentials/Week_2/world_7.png)

More generally, you have three types of key value pairs. Key value pairs for the input data, key value pairs for the intermediate data, and key value pairs for the output data. You can see a reference image in this slide. On top, you read input blocks of data. These blocks are processed by mappers and have intermediate key value pairs out of there. Then, data is aggregated by intermediate keys, and provided to reducers. Finally, data is transformed by reducers, and can be stored on local disks of a distributed system.

* Q: What was the reduce function on the slide you saw?
* A: "Max" function. Reducers transformations: (1,5) --> 5, (2,7) --> 7, (2,9,8) --> 9
* Q: What inferface do map and reduce functions have for a distributed "grep"?
* A: 
    * map: (docid, line) --> (docid, line*)
    * reduce: - (absent)
* Q: If you would like to group key-value pairs by key. How do you solve this task with Hadoop MapReduce?
* A:
    * map: (key, record) --> identity
    * reduce: identity --> [(key, [record])]
* Q: You are asked to count how many times each word occurs in a dataset.
* A:
    * map: (docid, line) --> [(word, 1), ...]
    * reduce: (word, [1,1,...]) --> (word, count)
    
### Fault Tolerance

HDFS is built of unreliable and cheap nodes. But any computational model on top of HDFS should be robust to failures.

In a distributed file system, you store information with duplication in order to overcome node failures. MapReduce framework should also provide robustness against node failures during the job execution.

![mr_fault](/Images/1_Big_Data_Essentials/Week_2/mr_fault.png)

When you spawn a MapReduce job, you will have one application master program that will control the execution. 
Master program will launch mappers to process input blocks or splits of data. 

To overcome the issues of correction execution mappers, you have to recollect functional programming benefits. There is no harm in re-executing mapper against the same data because you expect map function to be deterministic. As soon as you work on top of HDFS, you have a replica of this data on other nodes. So, you could assign another worker to a execute mapper against these data, and application master will do all this magic for you. You only have to provide an implementation of deterministic map function.

Quite a similar story with failures on a reduced side. 
If a worker running reducer dies, you can shuffle and sort data for this particular reducer to another worker. 
Application master is responsible for doing it. 
Shuffled and sorted data are stored on local disks instead of the distributed file system. 
It is intentional behavior because it is the intermediate data which should be used only during the limited amount of time. 
If at some point, this data is erased or somehow lost during the job execution, then due to functional properties, you can re-execute mapper on another alive worker to overcome obstacles.

* Q: Have you mentioned a single point of failure here?
* A: Application Master (If it dies, then the execution of the whole job is cancelled)

#### Hadoop MRv1

![MRv1](/Images/1_Big_Data_Essentials/Week_2/MRv1.png)

You have jobs and tasks:
* One MapReduce application is a job 
* A task can be either mapper or reducer (It is a map or reduce function applied to some chunk of data)

In the first version of Hadoop MapReduce framework, there was one global JobTracker to direct execution of MapReduce jobs. It is usually located on one high-cost and high-performance node with HDFS namenode. Just to remind you, namenode is responsible for the distributed file system metadata. 

TaskTrackers are located once per every node where you store data, or where datanode daemon is working.
TaskTracker spawns workers from mapper or reducer. It is easy to see that in this scenario, JobTracker is a single point of failure. That is why to reduce the load on JobTracker, some functionality of future versions was delegated to other cluster nodes.

#### YARN (Yet Another Resource Negotiation)

![yarn](/Images/1_Big_Data_Essentials/Week_2/Yarn.png)

There are no more TaskTrackers. 
They are substituted by NodeManagers who can provide a layer of CPU and RAM containers. 

ResourceManager overseas NodeManagers, and client request resources for execution from there. MapReduce applications can work on top of this resource layer. 

And at last, there is no concept such as a global JobTracker because application master can start on any node. All of these enable Hadoop to share resources dynamically between MapReduce and other parallel processing frameworks.

### Quiz

1) Select the type of failure when a server in a cluster gets out of service because of the HDD damages
    * Fail-Stop - True
    * Fail-Recovery
    * Byzantine failure
2) Select the type of failure when a server in a cluster reboots because of a momentary power failure
    * Fail-Recovery - True
    * Byzantine failure
    * Fail-Stop
3) Select the type of failure when a server in a cluster outputs unexpected results of the calculations
    * Fail-Recovery
    * Byzantine failure - True
    * Fail-Stop
4) Select the facts about Fair-Loss Link
    * Some packets are created out of nowhere
    * Some packets are lost regardless the contents of the message - True
    * Some packets are modified
5) Select the type of failure when some packets are modified during the transmission
    * Fair-Loss Link
    * Byzantine Link - True
6) Select a mechanism for capturing events in chronological order in the distributed system
    * Synchronize the clocks on all the servers in the system
    * Use a logical clocks, for example, Lamport timestamps
    * Both ways are possible, logical clocks hide inaccuracies of clocks synchronization - True
7) Select the failure types specific for distributed computing, for example for Hadoop stack products
    * Fail-Recovery, Fair-Loss Link and Asynchronous model - True
    * Fail-Stop, Perfect Link and Synchronous model
    * Byzantine failure, Byzantine Link and Asynchronous model
8) What phase in MapReduce paradigm is better to filtering input records without any additional processing?
    * Map - True
    * Shuffle & sort
    * Reduce
9) Select a MapReduce phase which input records are sorted by key:
    * Map
    * Shuffle & sort
    * Reduce - True
10) What map and reduce functions should be used (in terms of Unix utilities) to select the only unique input records?
    * map=’cat’, reduce=’uniq’ - True
    * map=’cat’, reduce=’sort -u’
    * map=’sort -u’, reduce=’sort -u’
    * map=’uniq’, reduce=’uniq’ - True
    * map=’uniq’, reduce=None
11) What map and reduce functions should be used (in terms of Unix utilities) to select only the repeated input records?
    * map=’uniq’, reduce=None
    * map=’uniq’, reduce=’cat’
    * map=’uniq -d’, reduce=’uniq -d’
    * map=’cat’, reduce=’uniq -d’ - True
12) What service in Hadoop MapReduce v1 is responsible for running MapReduce jobs:
    * TaskTracker
    * JobTracker - True
13) In YARN Application Master is...
    * A process on a cluster node to run a YARN application (for example, a MapReduce job) - True
    * A service to run and monitor containers for application-specific processes on cluster nodes
    * A service which processes requests for cluster resources