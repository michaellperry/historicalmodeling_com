---
layout: post
title:  "Guarantees"
date:   2011-01-28 12:00:00 -0600
categories: distributed-systems
permalink: /distributed-systems/guarantees.html
redirect_from: /book/distributed-systems/guarantees/
---
A distributed system based on durable message queues relies upon three guarantees:

- Messages will be **delivered** at least once.
- Message **duplication** will have no ill effects.
- Messages will be delivered in the **order** that they were sent.

As it turns out, these guarantees are difficult to ensure. Different distributed systems architectures have different strategies for upholding these guarantees.

## Delivery

In a distributed system based on message queues, at-least-once delivery is the easiest guarantee. The queue itself is durable. Once a message is queued, the sender relinquishes responsibility. Assuming that the recipient will eventually read from the queue (which our operations team monitors), the message will get delivered.

## Duplication

Duplication is a little trickier. It usually happens when the recipient has some trouble processing the message.

Queues don't protect us from duplication. After the recipient processes a message, it removes it from the queue. But if it can't finish its task, then it has to leave the message on the queue. Depending upon how much of its task was completed, some or all of the work may be duplicated the next time the recipient pulls the message.

One solution to duplication is idempotency. An **idempotent** message is one that will have the same outcome no matter how many times it is processed (assuming no other related messages intervene). Changing a customer's phone number is idempotent. Charging their credit card is not. Some messages can be designed to be idempotent, but not all.

To protect against duplication of non-idempotent messages, we have two strategies: journaling and transactions. A **journaling** strategy involves keeping track of the steps that have already been completed. We check the journal to ensure that we don't repeat those steps. A transaction-based strategy involves doing the work in the same transaction as the queue. A **distributed transaction coordinator** (DTC) ensures that both removing the message and completing the work happen as an atomic unit.

## Order

Order of delivery is the most difficult guarantee to uphold. One reason is poison messages.

When a recipient fails to process a message, it must leave the message on the queue. Otherwise messages will get lost (see the delivery guarantee). Most failures are **transient**, meaning that they are caused by temporary conditions, and might work if tried again. Deadlocks and timeouts are examples of transient failures. But some failures are intrinsic to the message itself. These **poison messages** will not succeed if retried. If they are left at the top of the queue, they will prevent later messages from being processed.

To detect a poison message, a service typically retries a specific number of times. Once that threshold is exceeded, the message is considered poison. The typical strategy for dealing with poison messages is to move them to a different queue. System operators monitor the poison message queue (also known as a **dead letter queue**) and intervene when messages arrive. They take whatever actions are necessary to ensure that the messages succeed, and then put them back on the application queue.

While this strategy allows the system to continue functioning, it changes the order in which the messages are processed. If a later message depends upon a poison message, then it will be processed in the wrong order. In the best case, the system detects the dependency and treats the later message as poison as well. In the worst case, results are undefined.

**Parallelism** can also cause messages to be processed out of order. If multiple nodes pull work from a single queue, there is no guarantee that they will finish that work in the same order that they started. If one of the nodes experiences a transient failure, the problem is exacerbated. While it is working on one message, other nodes will pull messages from further down the queue. If the first message fails, the service will put it back on the top to be processed later.

The most flexible solution to the ordering problem is to ensure that order between messages does not matter. Like idempotency, this can be achieved with most messages, but not with all. When order matters, the system must be programmed to recognize when it is violated. It can then move the later message to the bottom of the queue, thus increasing the likelihood that its prerequisite will be processed first.

Historical Modeling provides the three guarantees in the following ways:

- A fact is both **data and message**.
- **Identity** is determined by state.
- Predecessors define a **partial order** among facts.

## Data and message

Most architectures keep messages separate from the data. RPC messages are just wire protocols. Service busses store messages in queues, not in the database. Brokers persist the state of workflows separately, distinct from the data that those workflows operate on.

Historical Modeling, on the other hand, stores both data and message in facts. Facts store data, and can be queried to find the current state of an entity. Facts also represent messages, and can be queried to find work. When the user performs an action, a fact is stored. This implicitly sends the message.

Some distributed systems architectures rely upon a DTC to ensure that a message is only processed once. If the message fails, then the DTC rolls back both the removal of the message and the database update. But when the repository is the queue, a DTC is not necessary. Handling the message adds the fact that implicitly removes the message from the queue.

## Identity

A historical model takes advantage of immutability to protect against duplication. A historical fact cannot be modified. That immutable state identifies the fact. Any other fact with the same state **is the same fact**.

Suppose that a Stock fact has only one field: symbol. Any Stock where symbol=MSFT is the same fact. If we record a related fact (for example a Purchase by Account(12345) of 300 shares of Stock(MSFT) at 3:49pm on 1/28), then that related fact is also uniquely identified by its collection of fields. Another fact with exactly the same fields will be considered the same fact. It will not be duplicated.

## Partial order

Queuing systems attempt to impose a full order among messages. The queue itself does not know when that order is important and when it is not. As a result, it tries to uphold the order guarantee equally in all cases. A full order among messages is over-constrained.

Instead of trying to achieve full order, a historical model defines a partial order among messages. Each message has a reference to its predecessors. These messages must be sent first. The infrastructure knows this, and preserves order when necessary. It does not leave detection up to the application.

On the flip side, the infrastructure also understands when messages are unrelated. In those cases, it can freely violate the order guarantee. The application will not be adversely affected when unrelated messages are processed out-of-order.

A fact only references its direct predecessors. It does not directly reference all facts that were a part of the conversation. Nevertheless, the identity of a fact is dependent upon the identities of its predecessors. And that relationship is transitive. To understand the identity of a fact, a node must receive all of its direct and indirect predecessors. In this way, the predecessor relationships among the facts place them in the correct partial order relative to one another.

The queuing guarantees that distributed systems rely upon are not easy to achieve. Each architecture has its own mechanism for upholding these guarantees. Historical Modeling is no exception. It determines which guarantees are truly important to the correct operation of the system, and upholds them only when necessary.