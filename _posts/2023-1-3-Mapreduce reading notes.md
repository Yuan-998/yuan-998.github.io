---
layout: post
title: Reading Notes of MapReduce Paper
tags: [MapReduce, Distributed Systems]
readtime: true
---

## Introduction

This article is more about my understanding and questions towards MapReduce after finishing reading the paper and implementing the lab 1 of MIT 6.824.

All contents will be based on the following graph.
![Execution overview](../assets/img/MapReduce/MapReduce.jpeg)

### Input Files
It is mentioned in the paper that the input files will usually be split into `M` pieces of typically 16 megabytes to 64 megabytes per piece. My question is when it should be 16 megabytes and when it should be 64 megabytes. My understanding is that the split is actually logic split. So, the input files are untouched but the worker will read the specified amount of data from input files. Besides, the input files are stored in distributed file system, like `HDFS`. So, it would be more convenient for the worker to read files stored in itself to avoid remote read of data from other nodes bringing extra network cost. Hence, we may specify a split size that reduces remote read of data in distributed file system.

### Intermediate key/value pairs
One thing mentioned in the paper is that the intermediated key/value pairs produced by the `Map` function are buffered in memory and they will be written to local disk periodically after partitioned by the partitioning function. To my understanding, the purpose of this is to reduce the I/O workload by writing the pairs in batch. 

### Remote Read in Reduce Phase
In my implementation of lab 1 from MIT 6.824, I have implemented a strategy to have the worker in map phase to start reduce task when it finishes the map task. I am not sure whether this is really the case in MapReduce. But, I could see one benefit of doing so is that this can reduce the need for remote read. However, from the graph above, I would say the map workers and reduce workers are seperated.

### Master Failure
Unlike worker, the master is the one and only. So, when the master fails, when current implementation **aborts** the MapReduce computation.

### Output Files
Since the reduce task worker will write the results in many seperate files and sometimes we need to gather the results all together. In hadoop system, you can run the following command. `hadoop fs -getmerge /output/dir/on/hdfs/ /desired/local/output/file.txt`