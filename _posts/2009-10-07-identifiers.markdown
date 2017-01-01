---
layout: post
title:  "Identifiers"
date:   2009-10-07 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/identifiers.html
redirect_from: /book/examples/workitemtracker/identifiers/
---

Suppose we want the ability to find a Developer by name. Let’s define a fact that is unique based only upon name:

![Identifiers]({{ site.baseurl }}/images/identifiers.jpg)

```
fact DeveloperIdentifier
{
   string name;
}
```

If I try to create two DeveloperIdentifiers with the same name, I get the same object each time:

```
DeveloperIdentifier m1 = DeveloperIdentifier("mperry")
DeveloperIdentifier m2 = DeveloperIdentifier("mperry")
Assert(m1 == m2)
```

Now we define the DeveloperName not in terms of the string, but of the DeveloperIdentifier:

```
fact DeveloperName
{
   Developer developer;
   DeveloperName* prior;
   DeveloperIdentifier identifier;

   bool replaced { not exists DeveloperName next : next.prior = this }
}
```

This lets us write a query:

```
fact DeveloperIdentifier
{
   string name;

   Developer* developer
   {
      DeveloperName n : n.identifier = this where not n.replaced
      Developer d : n.developer = d
   }
}
```

This query gets all of the successor DeveloperNames (those that use this identifier) that have not been replaced. Then it gets all of their predecessor Developers. So this query lists all of the Developers having the given name.

We can use this query to look up a Developer by name:

```
Developer* developer = DeveloperIdentifier("mperry").developer
```

Because DeveloperIdentifiery(“mperry”) returns the same object every time, we will start the search from the right object. The search returns a list of developers currently having that name.

We could also use the “property” shorthand:

```
fact Developer
{
   unique;
   property DeveloperIdentifier identifier;
}

fact DeveloperIdentifier
{
   string name;

   Developer* developer
   {
      Developer d : d.identifier = this
   }
}
```

