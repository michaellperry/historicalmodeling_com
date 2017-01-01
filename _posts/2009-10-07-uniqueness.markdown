---
layout: post
title:  "Uniqueness"
date:   2009-10-07 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/uniqueness.html
redirect_from: /book/examples/workitemtracker/uniqueness/
---

First, we need to define top-level constructs:

![Uniqueness]({{ site.baseurl }}/images/uniqueness.jpg)

```
fact Project
{
   unique;
}

fact Developer
{
   unique;
}
```

Both a Project and a Developer are unique constructs in their own right. They both have “unique” fields (GUIDs, in practice) that make them unique. Without these fields, every Project would be the same, and every Developer would be the same.