---
layout: post
title:  "Properties"
date:   2009-10-07 12:00:00 -0600
categories: examples
permalink: /examples/work-item-tracker/properties.html
redirect_from: /book/examples/workitemtracker/properties/
---

Because the above pattern is so common in historical modeling, the Factual modeling language defines a construct for it. We could just write:

```
fact Project
{
   unique;
   property string name;
}
```

But it’s important to know what’s happening under the “property” keyword. Historical modeling does not allow things to change. It is using the above pattern to give the appearance of a mutable property.