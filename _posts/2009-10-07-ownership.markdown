---
layout: post
title:  "Ownership"
date:   2009-10-12 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/ownership.html
redirect_from: /book/examples/workitemtracker/ownership/
---

Whereas a developer can be a member of several projects over time, a work item belongs to only one project. It is part of that project when it is created, it cannot be a part of multiple projects, and it cannot be moved to another project. This is strict ownership.

Since all fields in a historical fact are immutable, ownership is simply represented as a field referencing the owner.
![Ownership]({{ site.baseurl }}/images/ownership.jpg)

```
fact WorkItem
{
   unique;
   Project project;

   property string description;
}
```

Besides the project, there are no other immutable attributes of a work item. Its description could change. Its type (defect, enhancement, user story, etc.) could change. It could be assigned to different developers over time. So we need the "unique" field to differentiate between work items within a project.

An owner usually knows about its children. This is accomplished with a query:

```
fact Project
{
   unique;
   property string name;

   WorkItem* workItems
   {
      WorkItem wi : wi.project = this
   }

   Developer* developers
   {
      ProjectMembership m : m.project = this
      Developer d : m.developer = d
   }
}
```

In this case, however, a query listing all of the work items in a project is probably not useful. There are better ways to organize things.