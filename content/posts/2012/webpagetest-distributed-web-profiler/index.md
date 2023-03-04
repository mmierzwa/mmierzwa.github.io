---
author: Marek Mierzwa
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-7340073300836604285
blogger_orig_url: http://byteloom.blogspot.com/2012/03/webpagetest-distributed-web-profiler.html
date: "2012-03-21T00:27:00Z"
tags:
- debugging
- web
- performance
title: WebPagetest - distributed web profiler
---

There are many popular tools for web performance profiling/debugging these days. From my personal tool set I could mention [Firebug](http://getfirebug.com/), IE Developers Toolbar (integrated with IE from version > 8) or [Fiddler](http://www.fiddler2.com/fiddler2/). The common problem with those tools is that they are installed and run locally on developers machine. Sometimes, for example when behaviour of the web app depends on loading all elements in the specific order or time, it is important to determine if those factors vary in different geographical locations. The download speed for China and Germany could be very different. If you face with this kind of problems I could recommend you a great distributed web performance profiler - [WebPageTest](http://www.webpagetest.org/).  


With this tool you can run a series of tests for a given URL, defined location (many available around the world) and browser. This is the most basic operation but you can choose also the visual performance comparison with other pages, test for mobile browsers or trace-route.  

For the basic web performance test you can specify (in advanced options) connection speed, number of test to repeat, network packet tracing (in tcpdump format for further analysis), authorization, adds blocking and many other stuff. You can script operations to perform on target page.  

In tests summary among standard metrics like loading time for scripts, styles, images and other elements, timing charts and content breakdown charts you will see a detailed HTTP requests/response report (with timing/offset not only for download itself but with DNS lookup or initial connection time). Next report tabs show a performance review with scoring and suggestions how you could improve you web page (by tuning some of the HTTP server settings like compression, utilizing CDN, caching or combining/minimizing JS/CSS). This feature should be familiar to all users of [PageSpeed](http://code.google.com/intl/pl/speed/page-speed/) (for Firebug as additional plug-in, for Apache as module etc.).  

In the end you can see how the user in previously defined location and web browser would see your page when it started loading and when the page has been fully loaded. This is available both on screen shots or in a captured video (the last one should be set in advanced options prior to the test). You can also save all the results in [HAR (HTTP Archive)](http://www.softwareishard.com/blog/har-12-spec/) or CSV format (and tcpdump if you selected the right option).  

In [the project documentation](https://sites.google.com/a/webpagetest.org/docs/advanced-features) I found some information about REST API and CLI for batch processing:  

> This command-line tool performs simple one-off batch testing. It loads the set of URLs in the input file as a batch, submits all of them to WebPageTest server which performs the tests, and then downloads the results of the successful tests and reports the failing tests. This tool is mainly implemented by the APIs in our batch processing library and hence can also serves as a sample usage of the batch processing APIs.

This sounds very interesting and I think I'll investigate this in future.  

[![Test settings panel - basic settings](main_advanced_1.png "Test settings panel - basic settings")](main_advanced_1.png)

[![Test settings panel - advanced settings](main_advanced_2.png "Test settings panel - advanced settings")](main_advanced_2.png)

[![Test settings panel - video capture for page rendering](main_advanced_3.png "Test settings panel - video capture for page rendering")](main_advanced_3.png)

[![Test results - summary for all test runs in test set](result_overview.png "Test results - summary for all test runs in test set")](result_overview.png)

[![Test results details - waterfall view](result_details_1.png "Test results details - waterfall view")](result_details_1.png)

[![Test results details - connection timings view](result_details_2.png "Test results details - connection timings view")](result_details_2.png)

[![Test results details - requests details](result_details_3.png "Test results details - requests details")](result_details_3.png)

[![Test results - performance review checklist](result_performance_review_1.png "Test results - performance review checklist")](result_performance_review_1.png)

[![Test results - performance review with scoring](result_performance_review_2.png "Test results - performance review with scoring")](result_performance_review_2.png)

[![Test results - performance review glossary](result_performance_review_3.png "Test results - performance review glossary")](result_performance_review_3.png)

[![Test results - Page Speed optimization check](result_pagespeed.png "Test results - Page Speed optimization check")](result_pagespeed.png)

[![Test results - content breakdown charts for the first page load](result_content_breakdown_1.png "Test results - content breakdown charts for the first page load")](result_content_breakdown_1.png)

[![Test results - content breakdown charts for the next page load](result_content_breakdown_2.png "Test results - content breakdown charts for the next page load")](result_content_breakdown_2.png)

[![Test results - content breakdown charts with domains specified](result_content_breakdown_by_domain.png "Test results - content breakdown charts with domains specified")](result_content_breakdown_by_domain.png)

[![Test results - page screen shots for the next page loading steps](result_screenshots.png "Test results - page screen shots for the next page loading steps")](result_screenshots.png)

[![Test results - page rendering video](result_grabvideo.png "Test results - page rendering video")](result_grabvideo.png)

[![Test results history. You can compare the results of multiple tests here](test_history.png "Test results history. You can compare the results of multiple tests here")](test_history.png)

Have a nice profiling with WebPageTest!

{{< blogspot_ref >}}