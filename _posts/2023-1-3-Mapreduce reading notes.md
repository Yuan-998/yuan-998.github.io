---
layout: post
title: Reading Notes of MapReduce Paper
tags: [MapReduce, Distributed System]
readtime: true
---

## Introduction

This article is more about my understanding and questions towards MapReduce after finishing reading the paper and implementing the lab 1 of MIT 6.824.

All contents will be based on the following graph.
![Execution overview](../assets/img/MapReduce/MapReduce.jpeg)

### MapReduce Abstraction
MapReduce system hides the messy details of `parallelization`, `fault-tolerance`, `data distribution`, `load balancing` and `scheduling` (not mentioned in the paper) in a library.

Maybe in the early stage of MapReduce, `scheduling` is not a part of the library and [it is introduced later](https://www.geeksforgeeks.org/hadoop-schedulers-and-types-of-schedulers/).

### Data Parallelism
concurrent execution of the same task on differnet data.

### Implementation
Different implementations of MapReduce for small shared-memory machine, NUMA multi-processor machine, and networked machines are possible.
### Input Files
It is mentioned in the paper that the input files will usually be split into `M` pieces of typically 16 megabytes to 64 megabytes per piece **automatically**. My question is when it should be 16 megabytes and when it should be 64 megabytes. My understanding is that the split is actually logic split. So, the input files are untouched but the worker will read the specified amount of data from input files. Besides, the input files are stored in Datanodes in distributed file system, like `HDFS`. So, it would be more convenient for the worker to read files stored in itself to avoid remote read of data from other nodes bringing extra network cost. Hence, we may specify a split size that reduces remote read of data in distributed file system.

### Intermediate key/value pairs
One thing mentioned in the paper is that the intermediated key/value pairs produced by the `Map` function are buffered in memory and they will be written to local disk **periodically** after partitioned by the partitioning function. To my understanding, the purpose of this is to reduce the I/O workload by writing the intermedia pairs in batch. 

### Remote Read in Reduce Phase
In my implementation of lab 1 from MIT 6.824, I have implemented a strategy to have the worker in map phase to start reduce task when it finishes the map task. I am not sure whether this is really the case in MapReduce. But, I could see one benefit of doing so is that this can reduce the need for remote read. See more [here](#jobtracker-and-tasktracker)

### Worker Failure
The master pings every worker periodically. If a worker doesn't response, any Map/Reduce tasks done (in case that those intermediate files are no longer accessible) or are being processed are marked as *idle* so that the tasks will be rescheduled.
### Master Failure
Unlike worker, the master is the one and only. So, when the master fails, when current implementation **aborts** the MapReduce computation.

### JobTracker and TaskTracker
Quote from [here](https://stackoverflow.com/questions/46684091/in-hadoop-what-is-the-difference-and-relationship-between-jobtracker-tasktracker) and modified.
**JobTracker**
> 1. JobTracker process runs on a separate node and not usually on a DataNode.
> 2. JobTracker is an essential Daemon for MapReduce execution in MRv1. It is replaced by ResourceManager/ApplicationMaster in MRv2.
JobTracker receives the requests for MapReduce execution from the client.
> 3. JobTracker talks to the NameNode to determine the location of the data.
> 4. JobTracker keeps track the state (*idle*, *in process*, *completed*) of each Map and Reduce task.
> 4. JobTracker finds the best TaskTracker nodes to execute tasks based on the data locality (proximity of the data) and the available slots to execute a task on a given node.
> 5. JobTracker [monitors](#worker-failure) the individual TaskTrackers and the submits back the overall status of the job back to the client.
> 6. JobTracker process is critical to the Hadoop cluster in terms of MapReduce execution.
> 7. When the JobTracker is down, HDFS will still be functional but the MapReduce execution can not be started and the existing MapReduce jobs will be halted.

**TaskTracker**
> 1. TaskTracker runs on DataNode. Mostly on all DataNodes.
> 2. TaskTracker is replaced by Node Manager in MRv2.
> 3. TaskTracker will be in [constant communication](#worker-failure) with the JobTracker signalling the progress of the task in execution.
> 4. Mapper and Reducer tasks are executed on DataNodes administered by TaskTrackers.
> 5. TaskTrackers will be assigned Mapper and Reducer tasks to execute by JobTracker.
> 6. TaskTracker failure is not considered fatal. When a TaskTracker becomes unresponsive, JobTracker will assign the task executed by the TaskTracker to another node.

### Output Files
Since the reduce task worker will write the results in many seperate files and sometimes we need to gather the results all together. In hadoop system, you can run the following command. `hadoop fs -getmerge /output/dir/on/hdfs/ /desired/local/output/file.txt`


### Backup Task
`straggler`: a machine that takes an unusually long time to complete one of the last few map or reduce tasks in the computation.

When a MapReduce operation is close to completion, the master schedules backup executions of the remaining in-progress tasks. The task is marked as completed whenever either the primary or the backup execution completes.

### Examples

#### Distributed Grep
```
map -> a line that matches the given pattern
reduce -> simply copy the output of map
```

#### Count of URL Access Frequency
```
map -> <URL, 1>
reduce -> <URL, total count>
```

#### Reverse Web-Link Graph
```
map -> <target, source> (each link to a target URL found in a page named source)
reduce -> <target, list(source)>
```

#### Term-Vector per Host
```
map -> <hostname, term vector>
reduce -> <hostname, term vector> (throwing away infrequent terms)

term vector = a list of <word, frequency>
```

#### Inverted Index
```
map -> <word, document ID>
reduce -> <word, list(document ID)>
```