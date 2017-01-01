---
layout: post
title:  "Audit Trail"
date:   2009-10-12 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/audit-trail.html
redirect_from: /book/examples/workitemtracker/audit_trail/
---

To track assignments of work items to developers, we'll create a folder concept.

![Audit Trial]({{ site.baseurl }}/images/audit_trail.jpg)

```
fact Folder
{
   unique;
   ProjectMembership membership;
   
   property string name;
}
```

Project membership is a prerequisite for creating a folder. A developer can create as many folders per project as he needs. A work item can be assigned to a folder, which implies that it is assigned to the associated developer.

The simplest way to assign a work item to a folder would be to create a property.

```
fact WorkItem
{
   unique;
   Project project;

   property string description;
   property Folder assignedTo;
}
```

A work item is assigned to a folder. That assignment can be changed at any time.

The problem with this solution is that it hides the history of folder assignments. An important part of tracking work items is to see their progress as they move from folder to folder. To make this more explicit, we'll express the full fact structure rather than using the "property" shorthand.

```
fact Assignment
{
   WorkItem workItem;
   Folder assignedTo;
   Assignment* prior;
   
   bool current
   {
      not exists Assignment next : next.prior = this
   }
}
```

This gives us the ability to explicitly query for either the current folder or the history of all assignments:

```
fact WorkItem
{
   unique;
   Project project;

   property string description;
   
   Folder* assignedTo
   {
      Assignment a : a.workItem = this where a.current
      Folder f : a.assignedTo = f
   }
   
   Assignment* assignments
   {
      Assignment a : a.workItem = this
   }
}
```

This also gives us a place to hang a note. A developer optionally enters a note when making the assignment, or any time while working on an assignment.

```
fact Note
{
   Developer by;
   Assignment assignment;
   string text;
}

fact Assignment
{
   WorkItem workItem;
   Folder assignedTo;
   Assignment* prior;
   
   bool current
   {
      not exists Assignment next : next.prior = this
   }
   
   Note* notes
   {
      Note n : n.assignment = this
   }
}
```

Properties hide the audit trail from the application. But by making properties explicit, we can both query and annotate the audit trail.

