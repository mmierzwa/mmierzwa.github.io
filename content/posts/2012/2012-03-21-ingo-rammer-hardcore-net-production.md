---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-4954829087087858226
blogger_orig_url: http://byteloom.blogspot.com/2012/03/ingo-rammer-hardcore-net-production.html
date: "2012-03-21T00:00:00Z"
tags:
- debugging
- .net
- il
title: Ingo Rammer - Hardcore .NET Production Debugging
---

Have you ever had a problem with a user complaining on and on that when he clicks some button in your application his computer hangs but it works fine on any other machine where you have tested it? The web service is consuming all the available memory in completely indeterministic way? Or maybe your web application is magically crashing on production machine when it works in the same usage scenario on development and test environment? If not, you are probably in about 1% of the most luckiest developers in the world (or you are not the developer).  

Last year I attended in [Norwegian Developers Conference 2011](http://www.ndc2011.no) in Oslo where among many other great presentations I've watched the one presented by Ingo Rammer - "Hardcore .NET Production Debugging".<!--more--> The [presenter](http://www.thinktecture.com/staff/ingo) is one of co-funders of consulting company [ThinkTecture](http://www.thinktecture.com) and specializes in profiling and debugging.  

Here is the presentation description from NDC2011 agenda:  

> **Hardcore .NET Production Debugging**  
> But ... it used to work yesterday! In this newest version of his classic session, Ingo Rammer will introduce the hardcore and low–level tools used for production debugging of .NET applications. You'll learn how to attack the nastiest bugs in your applications, how to look at what's causing that grinding halt of your ASP.NET application and how to find the cause of that horrible memory leak in your Windows Forms application. Knowledge of these production debugging tools like WinDbg and SOS is not only important for cases when you really don't have access to Visual Studio and your source code, but these tools also reveal a lot more information than just the regular managed code debuggers.

The presentation title speaks for itself. It looks really hardcore - watching this guy with WinDBG and other debugging tools from MS in action for the first time impressed me a lot. I recommend this to all .Net developers. No matter if you write web/desktop applications or system services you will find many advices that could save hours of your work, nerves and - priceless - the trust of your clients.  

You can watch the presentation on-line directly on NDC2011 page [here](http://ndc2011.macsimum.no/mp4/Day2%20Thursday/Track7%200900-1000.mp4).
