---
layout: post
title:  "The CAP Theorem"
date:   2011-02-08 12:00:00 -0600
categories: distributed-systems
permalink: /distributed-systems/cap-theorem.html
redirect_from: /book/distributed-systems/cap-theorem
---
Eric Brewer is an expert in distributed systems. In the Principles of Distributed Computing 2000 keynote address, he gave us the CAP Theorem. It states that a distributed system cannot simultaneously guarantee these three attributes:

- Consistency
- Availability
- Partition Tolerance

It can guarantee at most two. Which two you choose should depend upon the system’s architectural requirements.

## Consistency

![Consistency]({{ site.baseurl }}/images/consistency.png)

Consistency in a distributed system is not strictly the same as ACID consistency. A distributed system is consistent if a read at any node returns data that is no older than that written by a previous write. The read and the write may occur at the same node or at different nodes. The nodes may use any algorithm they wish to keep each other up-to-date. But if I write version 2, a consistent system will never again read version 1.

There are many ways to guarantee consistency. One would be to block writes until all nodes have been notified. Another would be to block reads until all nodes are consulted. Yet another is to delegate one node the master of that particular piece of data, and route all messages to it. In practice, consistent distributed systems use a combination of these algorithms.

## Availability

![Availability]({{ site.baseurl }}/images/availability.png)

A distributed system is available if any non-failing node responds to a request in a reasonable amount of time. It doesn’t mean that nodes can’t fail. It just means that, whatever other guarantees the system offers, it will respond when you address one of the remaining nodes.

You can see the tension between consistency and availability. To guarantee both, we need redundancy. What if the data that we try to read was only stored on the node that was lost after the write? That data would not be available, and we could not guarantee consistency.

## Partition Tolerance

![Partition tolerance]({{ site.baseurl }}/images/partition_tolerance.png)

A distributed system is partition tolerant if it can tolerate the loss of any number of messages. If enough messages are lost between islands of nodes, the network has been partitioned.

Network partitioning happens most often in wide area networks. A client disconnects from the internet. Or the connection between two data centers is severed. But network partitioning can happen in a local area network. No network, no matter how expensive, can guarantee that all packets are delivered. It is up to the designer of the distributed system to decide whether the system will tolerate message loss.

Most distributed systems respond to momentary message loss by resending messages. But since the network cannot guarantee message delivery, even those retries might be lost. It’s how the system responds to maintained message loss that determines whether it can guarantee partition tolerance.

## Proof

The [proof of the CAP Theorem](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf) is pretty simple. Informally, we can ask: how can a system guarantee both consistency and partition tolerance? If all messages between two given nodes are lost, how can a write at one affect a read at the other? No matter what algorithm you come up with, the only way to guarantee both consistency and partition tolerance is to give up availability. When messages between two nodes are lost, you must fail the read as if the second node was down. There is no way to respond with consistent data.

Or you can ask how a system can guarantee both consistency and availability. Remember our three example algorithms. If we block writes until every available node is notified, and those notifications are lost, then we must fail the write. If we block reads until every available node is consulted, and those messages are lost, we must fail the read. And if we delegate to the master, and that message is lost, then we fail as well.

And finally, we can guarantee both availability and partition tolerance, but we have to relax our consistency guarantee. Nodes might be down and messages might be lost, but if we assume that those problems will eventually be solved, then we can say that we will eventually be consistent. It is possible to read stale data from such a system, but given enough time all nodes will be up-to-date.

When designing a distributed system, consider the guarantees that the problem demands. But also consider the guarantees that you will be unable to make, and decide how best to respond in those situations.

## Eventual consistency

Most of the modern distributed systems frameworks have opted to relax the consistency guarantee. They instead promise “eventual consistency”, or that you will read the new value if you wait long enough. More formally, this guarantee states:

- All writes are durable.
- Once a version has been read, a later read will not return an earlier version.

All writes are durable. No data will be lost. Data might, however, be overwritten by a later write.

Once a write becomes visible at a particular node, that node will no longer return an earlier version. It will never go back in time to a state before that write completed.

![Eventual consistency]({{ site.baseurl }}/images/eventual_consistency.png)

Consider a timeline of writes. We collect that timeline fully ordered at the node where the writes occurred. Those writes produce a series of state changes. By observing the state at that node, we can detect whether a write took place.

Now, allow that stream of writes to move to another node. It causes a similar series of state changes there. If we observe the state at the second node, it might be earlier than the state at the first. Consistency is not guaranteed. But, once the second node catches up, it will not go back. It is eventually consistent.

## Historical Modeling

Like many other frameworks, Historical Modeling guarantees eventual consistency. It does so by transmitting historical facts from one node to another.

A historical fact is a record of a decision or state change that occurred at one node. All writes in a historical system are creations of new facts. Facts are never modified or destroyed.

If you observe the history of facts at a target node, you might find that the fact you just wrote at the source is not yet there. Consistency is not guaranteed. This allows the target node to remain available even if the network that it shares with the source is partitioned.

![Historical eventyal consistency]({{ site.baseurl }}/images/historical_eventual_consistency.png)

When the fact is eventually shared with the target node, the transmission includes its fields and predecessors. These fields and predecessors uniquely identify the fact and distinguish it from others. In this way, an observer can recognize the target fact as the same as the source.

Predecessor facts must be transmitted first, so predecessors will always be present. Successors, however, will arrive eventually. Once they do, they will never be deleted. An observer can query for successors to determine the current state of the system. He will find that the state will never go backwards to a time when the new facts did not exist.

