---
layout: post
title: Reading Notes of Lease
tags: [Lease, Distributed System, Consistency Protocol]
readtime: true
---

## Introduction

*leases* are proposed as a consistency protocol that handles host and communication failures using physical clocks

## Leases and Cache Consistency
A *lease* is a contract that gives its holder specified rights over property for a limited period of time.

Advantages of short lease term:
1. It can minimize the delay resulting from client and server failures
2. Short leases also minimize the false write-sharing that occurs. *False sharing* here means a lease conflict when no actual conflict in file access exists. 
3. Short lease terms reduce the storage requirements at the server, since the record of expired leases could be reclaimed.

## Read Lease and Write Lease
A read lease allows a node to read a piece of data, but not modify it. Read leases are typically granted for a longer period of time than write leases, since reading the data does not pose a risk of creating consistency problems.

A write lease, on the other hand, allows a node to both read and modify a piece of data. Write leases are typically granted for a shorter period of time than read leases, since modifying the data can create consistency problems if not properly coordinated. The owner node that controls access to the data will grant a write lease to only one node at a time, to ensure that only one node is making changes to the data.

The lease manager can evcit the leases when there is a conflict. **Evcited write lease must write back.** When a write lease is evicted, the node that held the lease is responsible for writing back any changes it made to the data to the owner node. This helps to ensure that the latest version of the data is available to other nodes that may request it. **Evicted read lease must flush/disable cache.** When a read lease is revoked, the node that held the lease must respond by discarding its cached copy of the data and re-fetching the latest version from the owner node. This helps to ensure that the node has an up-to-date copy of the data and that it does not cause consistency problems by accessing or modifying stale data.

## Choosing the Lease Term
The choice of leases term is based on the trade-off between `minimizing lease extension overhead` versus `minimizing false sharing`.

## Options for Lease Management
The client is free in deciding `when to request extension of leases`, `when to relinquish them`, and `when to approve a write`.

## Fault-Tolerance
Leases depend on well-behaved clocks.
```
Faster server clock / Slower client clock -> errors
Slower server clock / Faster client clock -> no errors but extra traffic
```

They can be detected by either a synchronization protocol or by including timestamps in lease-related messages.

## In-depth Analysis on Lease Timing
<a>https://zhuanlan.zhihu.com/p/268370901</a> (Written in Chinese)


## Questions
#### What is the lease in GFS?
For [GFS](2023-1-23-GFS.md), a lease is a period of time in which a particular chunkserver is allowed to be the primary for a particular chunk. Leases are a way to avoid having the primary have to repeatedly ask the master if it is still primary -- it knows it can act as primary for the next minute (or whatever the lease interval is) without talking to the master again.

#### Can a lease holder keep extending the lease?
In some cache consistency models, the lease holder may be able to continuously extend its lease as long as it remains available and responsive. However, this is not always the case, and the ability to extend a lease may depend on various factors such as system policies, resource constraints, and other conditions.

For example, the owner node may limit the duration of leases to prevent a single node from monopolizing access to a piece of data for an extended period of time. The owner node may also revoke leases if it detects that a node has failed or is no longer responsive, to ensure that stale data is not kept in the cache.

In some cases, the owner node may dynamically adjust the duration of leases based on various conditions, such as the current level of demand for a piece of data, the number of nodes requesting access to the data, and the amount of available resources.

Therefore, while the lease holder may be able to extend its lease in some cases, it is not always guaranteed and may depend on the specific cache consistency model and the policies implemented by the owner node.