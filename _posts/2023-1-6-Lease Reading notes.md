---
layout: post
title: Reading Notes of Lease
tags: [Lease, Distributed Systems, Consistency Protocol]
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