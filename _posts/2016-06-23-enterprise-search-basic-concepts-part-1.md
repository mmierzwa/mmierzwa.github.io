---
layout: post
title: Enterprise search - the first look
date: 2016-06-23 18:00:00 +0200
tags: [enterprise search, search]
---
Recently leafing through my notebook (yeah, I'm still using old-time paper) I found some ideas, quick-notes, self-brainstorming diagrams related with search systems. An original idea for this blog, at least for few first posts, was to present them. Still this requires introducing some of the basics of enterprise search and search systems in general (so I could spare some time in future just referring to those). This is what this post series is about.  

In this part I will try to answer two questions - why do people need an enterprise search systems and how complex they can be?<!-- more -->

## Little piece of history

I still remember the times when most of public the pages in the Internet were registered in the web version of [yellow pages](https://en.wikipedia.org/wiki/Yellow_pages) - the [web directories](https://en.wikipedia.org/wiki/Web_directory). Until the mid-90's most of the Internet portals was maintaining such service built manually or semi-automatically. Even if the content discovery was made by a crawler the content classification and final categorization was a human domain.  
Obviously when the number of resources in the Net was growing exponentially over the time and media other than text was becoming more and more popular the web directories were getting stale from day to day or even faster. The maintenance became a Sisyphean task.

Internet search engines are the response for the explosion of information on the web. They can discover and retrieve the information automatically and serve it in a way that no manually-built directory can - extremely fast, with enormous throughput and in intelligent way. While content retrieval and discovery seems to be obvious the efficiency of serving mechanism in comparison to web directories requires some explanation here. Having such a huge content database a simple key word matching or category browsing with sorting by date (or other attribute) is usually very time consuming and ineffective. With the search adjusted to the natural language and sorting results by relevance this task becomes manageable and convenient.

## What's the difference?

But how does it apply to the enterprise search? Internet search providers such as Google, Yahoo! or Bing deal with exabytes (1 exabyte = 10^18 byte) of publicly available content, mostly unstructured. Enterprise search systems are typically implemented in companies or other organizations of various size. They usually deal with much smaller amount of data (still too much for manually maintained databases). The most important differences are not in the amount but in it's nature: the data describes specific domains, it's usually structured or at least provided with some meaningful meta-data, in many cases it's not public (secure content).  
Another dimension of those systems is the source (or rather sources) and format of data - not only pages available on the web but also records from relational and non-relational databases, file shares, spreadsheets, other systems integrated with web services etc. Enterprise search in larger organizations typically allows for searching heterogeneous data.

## Concrete stuff

I will give two examples to picture how wide this area is and how many thing should be considered before implementing enterprise search in an organization.

**Documents/media/software library**. By this I mean all the kinds of systems designed for delivering digital version of user manuals, product documentation or any other form of written/recorded/drawn information or software packages in form of downloadable files. It's almost always decorated with rich meta-data like description, author, publication date, id, revision, languages and many other. Good search is essential not only in its full text capabilities but also for filtering and browsing. No matter if the audience is external customer or employee (or both) the chance is quite big that it will require security trimming so user would find only the results he is authorized to see. In many cases enterprise search will also help building a fast navigation ([faceted search](https://en.wikipedia.org/wiki/Faceted_search)) since many organizations describe their information in some kind of hierarchical classification system.  
In case when the attachments contain text it's findability is usually as important as for meta-data which brings the requirement for efficient parsing and indexing it also.

**Intranet portal**. They are more popular in larger organizations than the small ones therefore the number of multiple data sources which are required to be searchable is usually big and it's growing over the time. Most of the departments, business units or services want to be visible in this central point and commonly the search is the most important of its functionalities. Resources could be: intranet pages from multiple heterogeneous systems, classic CMSs, enterprise content management systems, [master data systems](https://en.wikipedia.org/wiki/Master_data), CRMs or even mail boxes and employees calendars (for personalized search).  
In addition to what I mentioned describing libraries the biggest challenge here is to merge results from all those sources in a way that make sense for the user. This includes not only technical aspects but also careful user experience (UX) design and testing. It's not easy to present such results as web pages, images, definitions, employee or customer data etc. together on a single page in a usable way.  
Other problem to solve is how to get the required data from sources and keep them up to date so the search results are not stale when they finally reach the user. Sometimes it's not possible or reasonable to fetch all the data from source systems or those systems prefer other kind of integration with their internal search systems. In this case it could be beneficial to use [federated search](https://en.wikipedia.org/wiki/Federated_search) approach instead.

I know that those examples definitely does not exhaust the list of possible applications but they rises some topics that I would like to continue in next posts. They are also cases from my professional life so they are not just some abstract concept.

## Wrapping up

I hope this post gave you some overview of the subject of enterprise search and maybe even some inspiration. Next one will be more technical - I will introduce more architectural concepts and describe the base building blocks of enterprise search systems.

Cheers!  
Marek
