---
layout: post
title:  "Distributed Systems"
date:   2011-01-05 12:00:00 -0600
categories: distributed-systems
permalink: /distributed-systems.html
redirect_from: /book/distributed-systems/
---
Distributed systems are enterprise solutions to large scale business problems running across multiple machines. They typically have several stakeholders, each from different divisions of an organization. Business run on distributed systems, and when the system fails, the business suffers.

The challenges in maintaining distributed systems are not just technical. They span several disciplines, including:

- Project management
- Business analysis
- Development
- Configuration management
- Operations
- Database administration
- Business intelligence

Major business problems include:

- Identifying the important metrics and indicators to make business decisions
- Aligning workflows with business processes
- Allocating ownership and funding for system development and maintenance

Major technical problems include:

- Getting relevant data to the appropriate system
- Reducing latency
- Guaranteeing business continuity in the face of technical outages
- Ensuring that no data or transaction is lost

Historical Modeling alone does not address all of these concerns. Instead, it works within a framework of thought developed by the brightest minds of the software industry, supported by past experience and ongoing research. This series of articles explores that framework, and the role that Historical Modeling plays within it.

## Mainstream guidance and tools

There are large gaps between expert thinking and common practice regarding distributed systems. Mainstream vendors like Microsoft, Oracle, and Force.com provide tools and platforms for a large set of solutions. Those tools and platforms are not intended specifically for distributed systems. When they are inappropriately applied, they often fail. Common failure scenarios include:

- Lost or duplicated transactions
- Inability to scale
- Fragility in configuration and operation
- Slow or unreliable reporting
- Misalignment of technical dependencies with business priorities

Mainstream guidance, inappropriately applied, is usually to blame for these failures.

For example, tools like Web Services and Windows Communication Foundation (WCF) lead us into building distributed systems exclusively on **remote procedure calls** (RPCs). RPCs are point-to-point: the caller knows about one specific recipient. RPCs are synchronous: the caller waits for the recipient to respond. RPCs are unreliable: if the call fails, the caller does not know whether the recipient has received the message. All of these factors conspire to cause lost transactions and fragile systems.

Additionally, tools like relational databases lead us to create a **system of record** (SOR). The SOR is the authority for all information pertaining to a specific topic. Having an SOR for each domain encourages us to ask the SOR every time we need information about that domain. This puts unnecessary load on the system, making it difficult to scale. Furthermore, when the SOR is unavailable, all downstream business is affected. When an important business system depends upon a less important system of record, technology is misaligned with business.

Finally, conventional enterprise development has taught us to create an **enterprise data model** (EDM). All of the data needed to run a business is stored in one place, normalized and indexed in one way, and completely interrelated. Updating an EDM in response to an application action sometimes imposes locks on several different tables to guarantee consistency. Reporting against an EDM requires that we join across many tables and run aggregate functions to get the necessary data for decision making. Taken together, this affects the scalability of our system, and the speed of our reports.

## Distributed systems theory and practice

None of these mainstream recommendations is incorrect in its own right. They simply cannot be broadly applied, particularly within a distributed system. Industry experts have known about the problems that inappropriate application causes. They have offered several solutions.

Rather than building distributed systems exclusively with RPCs, experts advise us to use message queues where appropriate. A well-placed message queue breaks the point-to-point coupling between components, leading to less fragile systems and better business alignment. It also creates reliable, transactional intermediate storage so that no messages are lost and the system is generally more reliable.

Instead of directly querying the system of record, experts advise that we should separate queries from commands. **Command query responsibility segregation** (CQRS) is the practice of creating two data stores per domain, one optimized for reads and the other optimized for writes. A background process moves data from the write side to the read side according to a service level agreement (SLA). This allows the system to scale, and still provide quick and reliable reporting.

Finally, rather than always updating state during a transaction, experts sometimes recommend **event sourcing**. This is the practice of recording an event stream, and using that stream as the source of knowledge. The event stream serves as an audit log, revealing every business operation that has occurred within the system. Furthermore, it serves as an authority. Any view of the system can be recreated by replaying the event stream. Correctly applied, event sourcing leads to more scalable and reliable systems that never loose or duplicate transactions.

## Continue reading

- [Message Queues]({{ site.baseurl }}{% post_url 2011-01-10-message-queues %})
- [Service Bus]({{ site.baseurl }}{% post_url 2011-01-14-service-bus %})
- [Guarantees]({{ site.baseurl }}{% post_url 2011-01-28-guarantees %})
- [The CAP Theorem]({{ site.baseurl }}{% post_url 2011-02-08-cap-theorem %})

## Resources

For more information on distributed systems and the experts who have influenced Historical Modeling, please see the following:

- [Udi Dahan – Clarified CQRS](http://www.udidahan.com/2009/12/09/clarified-cqrs/)
- [Greg Young – CQRS and Event Sourcing](http://codebetter.com/gregyoung/2010/02/13/cqrs-and-event-sourcing/)
- [Werner Vogels – Eventual Consistency](http://www.allthingsdistributed.com/2008/12/eventually_consistent.html)
- [Eric Brewer – The CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)
- [Eric Evans – Domain Driven Design](https://www.infoq.com/interviews/eric-evans-ddd-interview)
- [Martin Fowler – Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html)