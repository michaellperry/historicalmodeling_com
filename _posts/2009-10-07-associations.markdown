---
layout: post
title:  "Associations"
date:   2009-10-07 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/associations.html
redirect_from: /book/examples/workitemtracker/associations/
---

The last thing I want to do with Developers and Projects is to make a Developer a member of a Project. This gives them access to that Project’s tasks.  ![Associations]({{ site.baseurl }}/images/associations.jpg)

```
fact ProjectMembership
{
   Developer developer;
   Project project;
}
```

We can query for all of the projects that a Developer is a member of:

```
fact Developer
{
   unique;
   property DeveloperIdentifier identifier;

   Project* projects
   {
      ProjectMembership m : m.developer = this
      Project p : m.project = p
   }
}
```

Or all of the Developers who are members of a Project:

```
fact Project
{
   unique;
   property string name;

   Developer* developers
   {
      ProjectMembership m : m.project = this
      Developer d : m.developer = d
   }
}
```