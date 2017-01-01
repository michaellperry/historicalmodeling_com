---
layout: post
title:  "Work Item Tracker"
date:   down 13:01:00 -0600
categories: examples
permalink: /examples/work-item-tracker.html
redirect_from: /book/examples/workitemtracker/
---

![Work Item Tracker]({{ site.baseurl }}/images/workitemtracker.jpg)

A work item is a defect or an enhancement that a developer needs to perform in a software project. This model supports the following stories:

- A developer is a member of many projects. 
- A developer is identified by name.
- A developer has many folders within each project.
- A project has many work items.
- A work item is assigned to a developer's folder.
- A developer attaches a note to an assignment.
- A work item has a human readable identifier unique within the project.

The complete model expressed in [Factual]({{ site.baseurl }}{% link factual.md %}) appears below. The sections that follow walk through the design of this model.

```
namespace WorkItemTracker;

fact Project
{
   unique;
   property string name;

   // Find all developers who are members of this project.
   Developer* developers
   {
      ProjectMembership m : m.project = this
      Developer d : m.developer = d
   }
}

fact Developer
{
   unique;
   property DeveloperIdentifier identifier;

   // Find all projects of which this developer is a member.
   Project* projects
   {
      ProjectMembership m : m.developer = this
      Project p : m.project = p
   }
}

fact DeveloperIdentifier
{
   string name;

   Developer* developer
   {
      Developer d : d.identifier = this
   }
}

fact ProjectMembership
{
   Developer developer;
   Project project;
}

fact Folder
{
   unique;
   Membership membership;
   
   property string name;
}

fact Assignment
{
   WorkItem workItem;
   Folder assignedTo;
   Assignment* prior;

   // An assignment is current until it has been superseded.   
   bool current
   {
      not exists Assignment next : next.prior = this
   }
   
   Note* notes
   {
      Note n : n.assignment = this
   }
}

fact Note
{
   Developer by;
   Assignment assignment;
   string text;
}

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

   // Get the folder to which the work item is currently assigned.   
   Folder* assignedTo
   {
      Assignment a : a.workItem = this where a.current
      Folder f : a.assignedTo = f
   }

   // Walk the history of assignments, current or not.
   Assignment* assignments
   {
      Assignment a : a.workItem = this
   }
}

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

fact WorkItemIdentifier
{
   WorkItem workItem;
   Identifier identifier;
}

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

-[Uniqueness]({{ site.baseurl }}{% link _posts/2009-10-07-uniqueness.markdown %}) 

-[Names]({{ site.baseurl }}{% link _posts/2009-10-07-names.markdown %})

-[Queries]({{ site.baseurl }}{% link _posts/2009-10-07-queries.markdown %})

-[Properties]({{ site.baseurl }}{% link _posts/2009-10-07-properties.markdown %})

-[Identifiers]({{ site.baseurl }}{% link _posts/2009-10-07-identifiers.markdown %})

-[Associations]({{ site.baseurl }}{% link _posts/2009-10-07-associations.markdown %})

-[Ownership]({{ site.baseurl }}{% link _posts/2009-10-07-ownership.markdown %})

-[Audit Trail]({{ site.baseurl }}{% link _posts/2009-10-07-audit_trial.markdown %})

-[Unique Identifiers]({{ site.baseurl }}{% link _posts/2009-10-07-unique_identifiers.markdown %})