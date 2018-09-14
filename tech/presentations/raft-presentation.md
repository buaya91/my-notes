# Raft - Distributed Consensus Protocol

![img]()

-------

### Problem: Linearizability with replicated state machine?

![hoho](https://raw.githubusercontent.com/buaya91/image-storage/master/raft-presentation/linearizability.png)

-------

### Properties

1. Guarantee linearizability (ie. No stale read)
2. Single Leader Model, ie. throughput does not scale with no of nodes

-------

### Topology

![im](https://raw.githubusercontent.com/buaya91/image-storage/master/raft-presentation/raft-happy-path1.png)

-------

### Sub-Problems
1. Leader Election
2. Data Replication

-------

### Leader election

![img](https://raw.githubusercontent.com/buaya91/image-storage/master/raft-presentation/leader-election1.png)

-------

![](https://raw.githubusercontent.com/buaya91/image-storage/master/raft-presentation/leader-election2.png)

### How does follower decide to vote?

* Only 1 vote
* One vote for Candidate that is **UP_TO_DATE**
    - Term
    - Log Index
* If Candidate detect leader, convert to follower

-------

### Log Replication

![step1](???)

-------

![step1](???)

-------
![step1](???)

-------

### Log vs State

Log: Similar to event in event sourcing, eg. data that represent command coming from client
State: Similar to snapshot

-------

### How does follower decide whether to accept the log?

1. Leader is not from older Term.
2. Leader and Follower does not have conflict.
    - If there's conflict, follower will modify the log to reflect's Leader's log

-------

### Majority rule

* Leader requires majority votes
* Commit Log requires majority votes
* Read/Write only works when majority reach consensus

Majority rule helps 
* prevent divergence on Leader election
* ensure no stale read

-------

### Read Protocol

Same majority rule, only serves Read when majority agreed.

-------

### Write Protocol

1. Leader get consensus among majority
2. Commit
3. Return OK to client (OK guarantee linearizability)

-------

### Leader failed after commit, before return

1. Img illustration (Fail before commit)
2. Implications: client might make the same call twice
3. Solution: client request comes with unique id

