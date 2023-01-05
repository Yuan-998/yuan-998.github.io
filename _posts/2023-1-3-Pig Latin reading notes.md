---
layout: post
title: Reading Notes of Pig Latin
tags: [MapReduce, Distributed Systems]
readtime: true
---

This is the notes for reading the paper, *Building a High-Level Dataflow System on top of MapReduce: The Pig Experience*.

### The Limitations of MapReduce
- No support for complex N-step dataflows
- Lack of explicit support for combined processing of multiple data sets
- frequently-needed data manipulation primitives must be coded by hand

### What *Pig* System offers
> Our Pig system offers composable high-level data manipulation constructs in the spirit of SQL, while at the same time retaining the properties of Map-Reduce systems that make them attractive for certain users, data types, and workloads.

### System Overview
`Pig Lation Program -> One or more MapReduce jobs -> Execute the jobs on a given hadoop cluster`

#### Features
> - a step-by-step dataflow language where computation steps are chained together through the use of variables
> - the use of high-level transformations, e.g., `GROUP`, `FILTER`
> - the use of user-defined functions as first-class citizens

#### Three modes of user interaction
##### Interactive Mode
The user interacts with Pig through an shell (called *Grunt*) by typing Pig command. **The compilation and execution is triggered only when the user asks for output through SOTRE command**

##### Batch Mode
The difference between this mode and interactive mode is that the user can submit a script containing Pig commands.

##### Embeded Mode
Pig offers a Java library. This option permits dynamic construction of Pig Latin programs, as well as dynamic control flow.

#### Command Transformation
![Compilation_and_execution](../assets/img/Pig/Pig_compilation_execution.png)

##### Parsing
The parser checks the syntactical correctness of the program, validity of all referenced variables, type, schema inference, and other checks.
The output of the parser is a logical plan organised in a directed acyclic graph with a one-to-one correspondence between Pig Latin statements and logical operators

##### Logical Optimizer
In this stage, the logical operators will be optimized, such as [projection pushdown](https://towardsdatascience.com/predicate-vs-projection-pushdown-in-spark-3-ac24c4d11855). Then they will be compiled into a series of Map-Reduce jobs

##### Map-Reduce Optimizer
Optimization based on Map-Reduce *combiner* stage to performa early paritcal aggregation

In the end, the DAG of optimized Map-Reduce jobs is toppologically sorted and ready to be executed by Hadoop in that order.

### Type System and Type Inference
Standard scalar types: `int, long, double and chararray (string), and bytearray`
Complex types: `map, tuple, and bag`
- map: `key - chararray, value - any type`
- tuple: an order list of data elements (can be any types, even nesting of complex types)
- bag: a collection of tuples

#### Type Declaration
1. no data types are declared. Treat all files as `bytearray` in this case. If the program uses an operator that expects a certain type on a field, then Pig will coerce that field to be of that type. Another case where Pig is able to know the type of a field even when the program has not declared types is when operators or user-defined functions (UDFs) have been applied whose return type is known.

2. declare types explicitly during LOAD as part of the AS clause

3. declaring types is for the load function itself to provide the schema information, which accommodates self-describing data formats such as JSON.

#### Lazy Conversion
When Pig does need to cast a `bytearray` to another type because the program applies a type-specific operator, it delays that cast to the point where it is actually necessary.

### Compilation to Map-Reduce
From logical plan to Map-Reduce execution plan

#### Logical Plan Structure
![Logical_plan](../assets/img/MapReduce/logical_plan.png)

#### Map-Reduce Execution Model
![Exexcution_model](../assets/img/MapReduce/execution_model.png)

1. Map Stage. `Map` stage process the raw input data one by one and produces a stream of data items annotated with keys.
2. Local Sort. To sort the data from map stage by key
3. Combine. This is an optional stage. It is for partial aggregation.
4. Shuffle Stage. Redistribute data among manchines to achieve a global organization of data by key.
5. Merge/Combine. A single ordered stream in merge stage and a possible combiner after each intermediate merge step.
6. Reduce Stage. Process the data associated with each key in turn, often performing some sort of aggregation.

#### Logical-to-Map-Reduce Compilation
Pig first translates a logical plan into a physical plan, and then embeds each physical operator inside a Map-Reduce stage to arrive at a Map-Reduce plan.
![logical_plan_to_mapreduce_plan](../assets/img/MapReduce/logical_plan_to_mapreduce_plan.png)

##### Logical Plan -> Physical Plan
`GROUP` -> `local rearrange, global rearrange, and package`. Rearrange means either hashing or sorting by key.

`JOIN` -> (1) `COGROUP` followed by a `FOREACH` operator to perform "flattening" (2) fragment-replicate join

##### Physical Plan -> Hadoop stages
Local rearrange operator simple annotates tuples with keys and stream identifiers, and let the Hadoop local sort stage do the work.
Golbal rearrange operators are removed because their logic is implemented by the Hadoop shuffle and merge stage.
Load and store are also removed, because the Hadoop framework takes care of reading and writing data.

### Map-Reduce Optimization and Job Generation
Only one optimization is carried at this level: Pig breaks *distributive* and *algebraic* aggregation functions into a series of three steps: *initial*, *intermediate*, and *final*. These steps are assigned to the `map`, `combine`, and `reduce` stages respectively.

The final step generates a Java jar file that contains the Map and Reduce implementation classes, as well as any user-defined functions that will be invoke as part of the job.

## Plan Execution