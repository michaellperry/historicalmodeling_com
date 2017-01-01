---
layout: post
title:  "Unique Identifiers"
date:   2009-10-23 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/unique-identifiers.html
redirect_from: /book/examples/workitemtracker/uniqueidentifiers/
---

Work items need unique identifiers within a project. These aren't required by the model, since the model uses the "unique" field. Instead, these are required by people talking about and exchanging email regarding work items. They are also used to reference work items in external systems that are not connected to the historical model. This unique identifier has to be human readable.
![Unique Identifiers]({{ site.baseurl }}/images/uniqueidentifiers.jpg)

Any client can create a work item without deferring to a central authority. The client cannot assign a unique identifier to the work item. Until the client synchronizes, the work item remains anonymous. After it synchronizes, a centralized work item identification service can assign it a unique identifier.

**Work item identification service**

We install a centralized service running on one machine. This service claims a project for which it will generate identifiers. By mutual agreement, no other service can claim that project. This is not enforced by the model, just by convention.


The work item identification service has a proxy fact that exists in the historical model. From this proxy fact, it can query for work items that need identifiers.

```
fact WorkItemIdentificationService
{
   Project project;

   // Find the work items that need identifiers.
   WorkItem *unidentifiedWorkItems
   {
      WorkItem wi : wi.project = this.project
         where not exists wi.identifier
   }
}
```

For the first unidentified work item that it finds, the identification service creates a unique identifier.

```
fact Identifier
{
   Project project;
   string identifier;

   // Get the work item for this unique identifier.
   // This will be assigned by an identification service.
   WorkItem* workItem
   {
      WorkItemIdentifier wiid : wiid.identifier = this
      WorkItem wi : wiid.workItem = wi
   }
}
```

It then assigns this identifier to the work item via an associative WorkItemIdentifier fact.

```
fact WorkItemIdentifier
{
   WorkItem workItem;
   Identifier identifier;
}
```

The work item can then query for its identifier through this association.

```
fact WorkItem
{
   unique;
   Project project;

   property string description;

   // Find the unique identifier of this work item.
   // This will be assigned by an identification service.
   Identifier* identifier
   {
      WorkItemIdentifier wiid : wiid.workItem = this
      Identifier id : wiid.identifier = id
   }

   // ...
}
```

When the work item is first created, it has not yet been given a unique identifier. The *identifier* query will be empty, so the work item will appear in *unidentifiedWorkItems*. After the identification service has given it an identifier, the *identifier* query will no longer be empty. At that point the work item will no longer appear in *unidentifiedWorkItems.* The query acts as a queue.

