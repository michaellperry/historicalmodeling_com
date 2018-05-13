---
title: A Learning Journey
layout: post
permalink: /Journey/
date: 2018-04-28
author: Jan
order: 0
---

## Historical Modeling: from Pluralsight to Production

Hi, my name is Jan.  I'm a developer from Belgium who is, maybe just like you, looking for solutions that really work in the area of mobile applications.

For a while now I have been fascinated by Michael's promising concept of Historical Modeling. I promised Michael to write a series of articles, which will be published on this blog over the next couple of months. In this series you can follow me on my journey discovering how to make good use of Historical Modeling in order to serve customers needs more efficiently.

### Let's start with some context

A couple of years ago I developed a mobile Android application for field-workers.  For dispatching I made a Windows application (the console) that allowed the dispatchers to plan and send tasks to the workers mobile phones.  The workers could view their planning and register their activities on their phones, including parts and materials, which were then synchronised back to the dispatcher for further processing and eventually for invoicing.

This setup was build around a central SQL Server Express database running on a hosted server.  The console (the dispatcher's Windows application) accessed this SQL database directly over the internet.  The workers' phones had their own local database (SQLite) which synchronized with the SQL Server database using a 3G/4G internet connection.  This synchronisation was built on Microsoft's [Sync Framework](https://msdn.microsoft.com/en-us/library/bb902854(v=sql.110).aspx) toolkit in combination with an old version of the open sourced [SyncFrameworkAndroid](https://github.com/SelvinPL/SyncFrameworkAndroid) made by [Selvin](https://github.com/SelvinPL).

The system is running fine in production with several customers, however as the customers base is increasing and customers keep asking for enhancements, I'm running into troubles with this setup.

### A few of the problems/limits I encountered

- Although the Sync Framework Toolkit v4.0 was open sourced by Microsoft, the Sync Framework core library, which contains the magic, is still closed source and Microsoft seems to have abandoned the project.
- The Sync Framework is synchronising state, which makes the synchronisation process rather complex with a lot of edge cases.
- Every schema change must be implemented as well on the server as on the apps.  This results in a very tight coupling and hence a very difficult process each time I want to install a new feature that requires a database schema change.
- The workers register their activities on their mobile phone by simply pushing the corresponding icon (work, transport, pause, private, ...) the moment they start the activity.  However sometimes they forget to push, or they push the wrong icon, and hence they have to correct the registration afterward.  For traceability this correction should not override the original, so we have to keep history.  Did someone mention "Temporal Databases"?  Temporal databases is relatively new and relatively complicated stuff.  What if we combine this with the complexity of state-based synchronisation?  I don't even want to think about it!
- So workers can apply minor corrections them self, however if the correction is more complicated it might be more efficient to call the dispatcher and let him handle the correction on his big screen.  But what happens if the worker and the dispatcher both apply conflicting corrections when the mobile phone is offline?  Indeed, conflicts!!
- So far for lookup-tables (customers, materials, locations, ...), if the data changes, we override the current version with the new version.  I know, a lot of system work that way, but if we want to do it correctly and avoid nasty surprises, we have to keep history.
- Having the console access the central SQL Server database directly has 2 major negative consequences:  1. The console needs a reliable internet connection, so mobile usage is not possible.  2. The console puts a heavy burden on the server and on the internet connection, hence scalability is limited.
- SQL Server Express is free but it is limited (e.g. database size).  Once your system outgrows these constraints, your in for a heavy license-fee.

So these are some of the issues I want to be solved. In my next episode we will get into a new architecture based on Historical Modeling and see how that can improve the system.

In meanwhile if you want a short introduction to Historical Modeling, start with the video [What is Historical Modeling](https://www.youtube.com/watch?v=ptVJTrJ8mQE). If however you prefer a deep dive, I recommend you start with Michael's course at PluralSight: [Occasionally Connected Windows Mobile Apps: Collaboration](https://app.pluralsight.com/library/courses/occasionally-connected-windows-mobile-apps-collaboration/table-of-contents).
