---
layout: post
title:  "Service Bus"
date:   2011-01-14 12:00:00 -0600
categories: distributed-systems
---
Message queues can improve the reliability and scalability of a distributed system when carefully applied. They solve the RPC problems of synchronous and unreliable messaging. However, they do not solve the problem of one-to-one coupling and configuration. The sender knows which queue to push messages to, and the recipient knows which queue to pull messages from. Service busses solve that problem.

A service bus is not a single monolithic component. Nor is it an infrastructure stack running on a cluster of machines (for example, BizTalk). Those are brokers. A service bus is a logical relationship among distinct services running on different machines, known as **nodes**. Often, those services use the same framework as one another, but that framework is installed individually at each node. Some popular service bus frameworks for .NET are:

- [NServiceBus](http://www.nservicebus.com/)
- [Rhino Service Bus](http://ayende.com/Blog/archive/2008/12/17/rhino-service-bus.aspx)
- [MassTransit](http://masstransit-project.com/)

## Messages

The most important task in designing a distributed system is defining the right set of **messages**. A message is an immutable block of information. It contains all of the information needed for the recipient to do something meaningful. Messages typically represent one of two things:

- **Command** – a request for the system to take action
- **Event** – a notification that something has occurred related to the business domain

Commands generally flow from the user of the system. A command might be "Submit order", or "Accept payment". Events, on the other hand, flow from one part of the system to another. An event might be "Order submitted", "Order shipped", or "Payment received". Events are named with a past tense verb, while commands have an imperative verb.

A **conversation** is a series of related messages. The messages are all about the same thing (perhaps an order or a patient visit). The messages have a cause-and-effect relationship among them. A command message comes from the user and kicks off the conversation. Then the recipient of that command makes some decision, takes some action, and publishes one or more event messages. Other nodes respond to those events and the conversation continues. Conversations may end quickly, or they may continue for very long periods of time.

## Handlers

![Command handlers]({{ site.baseurl }}/images/handlers.png)

A service bus delegates message processing to **handlers**. A handler is a service running on a node that responds to a single kind of message.

There is typically only one handler for each type of command message. When the user issues a "submit order" command, one service is responsible for validating it and entering it into the database. Many nodes may be competing for that command, but only one will be selected to perform it.

On the other hand, there can be many handlers for event messages. Events in a problem domain have lots of side-effects. When an order is submitted, an invoice must be sent, items must be picked and shipped, and customer preference must be adjusted. Each of these side-effects is a separate handler for the "order submitted" event.

Handlers are not coupled to each other. They only know about the types of messages they consume and create. In fact, handlers are not even coupled to their queues. The service bus determines which queue a handler consumes messages from, and which other queues it posts messages to. This is all based on configuration.

## Configuration

Like I said earlier, a service bus is not one monolithic thing. As such, there is no central configuration. Instead, each node is configured with the queue names and handlers that it needs to play its part in the system.

The user interface (typically a web server) is responsible for sending command messages to the proper handler. Therefore the UI node is configured with the name and location of the queue that each command handler pulls from.

![SubmitOrder command]({{ site.baseurl }}/images/submit_order_command.png)

The command handler's node is configured to pull messages from that same queue. But it is not configured with the names and locations of downstream queues. Remember that the command handler is creating event messages, and events have multiple handlers. If each command handler were configured with every downstream event handler, the operational overhead would be unbearable.

![SubmitOrder handler]({{ site.baseurl }}/images/submit_order_handler.png)

Instead, event handlers **subscribe** to their messages. Command handlers **publish** messages. Subscribers tell the bus which messages they want to receive. The bus is configured with the names and locations of publishers of those messages. The bus registers with those publishers, so that the publishers know where to queue messages without the need of explicit configuration.

![OrderSubmitted event]({{ site.baseurl }}/images/order_submitted_event.png)

To summarize, a service bus does two things:

- Decouples handlers from queues
- Adds multicast

Message queues are infrastructure components that must be explicitly provisioned. Without a service bus to route the messages, each service would have to know which queues to pull from and push to. Because a message is consumed as soon as it is processed, a single message queue does not support multicast. The service bus implements multicast patterns on top of queues.

## Service bus in a historical system

In historical modeling, every fact is potentially a queue. This means that a queue is logical, not physical. No provisioning of infrastructure is required to set up a queue. While this doesn't eliminate the need for a service bus, it does change the nature of that need.

In a historical model, a **fact** plays the part of a message. A fact is a historical record of a decision made either by a user or by the system. A fact can represent both a command and an event.

To begin a historical conversation, the user interface creates an initial fact. This fact acts as both a command and an event. This fact is typically named with a noun. The verb ("submit", "process", etc.) is implied. For example, the UI would create the "Order" fact.

![Order fact]({{ site.baseurl }}/images/order.gif)

The Order is published to the Company. Any node that subscribes to the company will receive the Orders.

```
fact Order {
    publish Company company;
    Customer customer;
    OrderLine* items;
}
```

A historical model acts as both message queue and database. As a result, it is not necessary to create a command handler to write the order to the database. It's already there. Instead, we can focus on the event handlers. Each fact handler is responsible for one side-effect. For example, one handler will ship the order, and another will prepare an invoice. Each side-effect is itself a fact.

![Invoice and Shipment facts]({{ site.baseurl }}/images/invoice_shipment.gif)

Instead of pulling facts from a physical queue, a handler runs a **query**. The query returns all facts to which the side-effect has not yet been applied. For example, the invoicing service processes all orders that have not yet been invoiced.

```
fact Company {
    unique;

    Order* ordersPendingInvoice {
        Order o : o.company = this
            where o.isPendingInvoice
    }
}
```

When the handler completes its task, it adds the Invoice fact. Adding this fact saves the information to the historical database. But, it also publishes the event for any down-stream handlers (accounts receivable or collections, for example). Furthermore, it effectively removes the Order from the `ordersPendingInvoice` query. This is accomplished through the `isPendingInvoice` **predicate**.

```
fact Order {
    publish Company company;
    Customer customer;
    OrderLine* items;

    bool isPendingInvoice {
        not exists Invoice i : i.order = this
    }
}
```

A service bus in a historical system does not need to route messages to the correct queues. Instead, a historical service bus invokes handlers based on logical subscriptions. Each handler subscribes to a root fact. When subsequent facts are published to that root, the handler is notified. The handler then executes the appropriate query and processes the facts. Its response removes the fact from the query.