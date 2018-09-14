# Distributed system

What is Distributed System in Computer Science?

A system is considered distributed if
* It contains of multiple processes
* Processes communicate via messaging
* Time taken for message to reach other parties is significant
compared to time between events, eg. if events only happen once per day,
even if communication takes 2 minutes, it's unnoticeable, and thus we
can model the system as non-distributed

In practice, we normally refer spatially distributed system as distributed
system, eg. system with multiple nodes

## Problems in distributed system

### 1. Failures, failures, failures

In a distributed system, Nodes are connected by network, unsurprisingly,
both can fail, when design such system, one need to design how to deal
with such failure, for more, read **CAP Theorem**.

### 2. Synchronization, because clock drift

A main problem in distributed system is how does each processes in the system
reach consensus, ie. how do they synchronize.

As every process is capable of changing the state of the system, ordering of
operations/events matter a lot. Human have been using clock time to understand
ordering of events, this however does not work in computer system as system
are known to susceptible for clock drift, as most system rely on NTP.

### 3. Distributed Transactions

How to achieve transactional behavior in distributed system ?

## Motivation

Why distributed system?

* More nodes -> More computing power
* Redundancy for failover
* Internet, people want to get connected despite the physical barrier

## Interesting concepts

### Raft algorithm
An algorithm that allow nodes in a distributed system to reach
consensus in a deterministic way, core idea includes

* Only 1 leader/master
* Consensus reached by agreement of more than half of the nodes
* Rely on persistent storage to support node failure and reboot

Available as long as more than half of nodes are working.

It works by deterministically elect a leader, and let leader to get
agreement with other nodes.

Pros and cons: TODO

### Two-phase commit

An algorithm to coordinate a number of nodes for certain operation/update.
Typically used for database.

Core idea is to split an operation into 2 phase. It requires a
coordinator to initiate each phases, which it can be a single point of
failure and require extra mechanism to prevent

* Prepare: Prepare for operations, nodes will promise to be able to commit the operations
* Commit: Commit the operation as promised

It solves the problem some nodes failed when broadcasting operations
by first confirming every node can commit.

### Lamport clocks

Lamport's clock is a kind of logical clock that allows distributed system
to deduce casual relationship between processes. It works in a distributed
message passing system.

It uses a concept of `happen-before` which is a `causal relationship`

It requires all processes to

* Tag all events with a logical timestamp that is stored by corresponding process that allow comparison, eg. Int
* On single process, if event A happens before event B, event B should have a greater timestamp value
* Sending event message and receiving are both events
* Increment timestamp after every event
* When receive message, update timestamp to the `max(msg's ts + 1, current timestamp)

By doing the above, we can know a few things

1. If `a -> b`, then T(a) > T(b)
2. If `T(a) > T(b)`, we cannot guarantee `a -> b`, but we can be sure
that `b -> a` is false

### Vector clocks

Vector clock is another kind of logical clock, it differs from Lamport's
clock by storing timestamps of all processes in the logical clock.

The logical clock is a vector of `(processId -> timestamp)` pair, thus
the name vector clock.

The rules are as follow

* All client starts with it's own clock with some default value X0
* Any event should trigger the clock value to increase by 1
*



## References
* https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing
