---
layout: post
title: WebPagetest - distributed web profiler
date: 2012-03-21 00:27 +0100
author: Marek Mierzwa
tags: [debugging, web, performance]
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-7340073300836604285
blogger_orig_url: http://byteloom.blogspot.com/2012/03/webpagetest-distributed-web-profiler.html
---

There are many popular tools for web performance profiling/debugging these days. From my personal tool set I could mention [Firebug](http://getfirebug.com/), IE Developers Toolbar (integrated with IE from version > 8) or [Fiddler](http://www.fiddler2.com/fiddler2/). The common problem with those tools is that they are installed and run locally on developers machine. Sometimes, for example when behaviour of the web app depends on loading all elements in the specific order or time, it is important to determine if those factors vary in different geographical locations. The download speed for China and Germany could be very different. If you face with this kind of problems I could recommend you a great distributed web performance profiler - [WebPageTest](http://www.webpagetest.org/).  
<!-- more -->

With this tool you can run a series of tests for a given URL, defined location (many available around the world) and browser. This is the most basic operation but you can choose also the visual performance comparison with other pages, test for mobile browsers or trace-route.  

For the basic web performance test you can specify (in advanced options) connection speed, number of test to repeat, network packet tracing (in tcpdump format for further analysis), authorization, adds blocking and many other stuff. You can script operations to perform on target page.  

In tests summary among standard metrics like loading time for scripts, styles, images and other elements, timing charts and content breakdown charts you will see a detailed HTTP requests/response report (with timing/offset not only for download itself but with DNS lookup or initial connection time). Next report tabs show a performance review with scoring and suggestions how you could improve you web page (by tuning some of the HTTP server settings like compression, utilizing CDN, caching or combining/minimizing JS/CSS). This feature should be familiar to all users of [PageSpeed](http://code.google.com/intl/pl/speed/page-speed/) (for Firebug as additional plug-in, for Apache as module etc.).  

In the end you can see how the user in previously defined location and web browser would see your page when it started loading and when the page has been fully loaded. This is available both on screen shots or in a captured video (the last one should be set in advanced options prior to the test). You can also save all the results in [HAR (HTTP Archive)](http://www.softwareishard.com/blog/har-12-spec/) or CSV format (and tcpdump if you selected the right option).  

In [the project documentation](https://sites.google.com/a/webpagetest.org/docs/advanced-features) I found some information about REST API and CLI for batch processing:  

> This command-line tool performs simple one-off batch testing. It loads the set of URLs in the input file as a batch, submits all of them to WebPageTest server which performs the tests, and then downloads the results of the successful tests and reports the failing tests. This tool is mainly implemented by the APIs in our batch processing library and hence can also serves as a sample usage of the batch processing APIs.

This sounds very interesting and I think I'll investigate this in future.  

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://3.bp.blogspot.com/-Y83WlwFqCik/T2upMGgXRKI/AAAAAAAABNA/zZaYNGBmPa0/s320/main_advanced_1.PNG)](http://3.bp.blogspot.com/-Y83WlwFqCik/T2upMGgXRKI/AAAAAAAABNA/zZaYNGBmPa0/s1600/main_advanced_1.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test settings panel - basic settings</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://2.bp.blogspot.com/-R3Cahbe7GL4/T2upOMJG0_I/AAAAAAAABNI/WHXojdvM-uo/s320/main_advanced_2.PNG)](http://2.bp.blogspot.com/-R3Cahbe7GL4/T2upOMJG0_I/AAAAAAAABNI/WHXojdvM-uo/s1600/main_advanced_2.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test settings panel - advanced settings. Here you can set for example tcpdump capturing</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://2.bp.blogspot.com/--8zu9A6OYwY/T2upPgvGReI/AAAAAAAABNQ/y3vjiSvdtEs/s320/main_advanced_3.PNG)](http://2.bp.blogspot.com/--8zu9A6OYwY/T2upPgvGReI/AAAAAAAABNQ/y3vjiSvdtEs/s1600/main_advanced_3.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test settings panel - video capture for page rendering</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://3.bp.blogspot.com/-Ecu1CMsv_6Y/T2upfuYvRYI/AAAAAAAABOQ/e8c_ZHlV3TA/s320/result_overview.PNG)](http://3.bp.blogspot.com/-Ecu1CMsv_6Y/T2upfuYvRYI/AAAAAAAABOQ/e8c_ZHlV3TA/s1600/result_overview.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - summary for all test runs in test set</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://2.bp.blogspot.com/-9Ailb_Y1cSo/T2upW08utJI/AAAAAAAABNw/QlL9bR4gloU/s320/result_details_1.PNG)](http://2.bp.blogspot.com/-9Ailb_Y1cSo/T2upW08utJI/AAAAAAAABNw/QlL9bR4gloU/s1600/result_details_1.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results details - waterfall view</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://3.bp.blogspot.com/-i0u6zDvH6U0/T2upY4Eim_I/AAAAAAAABN4/GZJpkRV_dsY/s320/result_details_2.PNG)](http://3.bp.blogspot.com/-i0u6zDvH6U0/T2upY4Eim_I/AAAAAAAABN4/GZJpkRV_dsY/s1600/result_details_2.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results details - connection timings view</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-APdAE8V3jK8/T2upauUl69I/AAAAAAAABOA/tVr7eP6iAYw/s320/result_details_3.PNG)](http://4.bp.blogspot.com/-APdAE8V3jK8/T2upauUl69I/AAAAAAAABOA/tVr7eP6iAYw/s1600/result_details_3.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results details - requests details (times for DNS lookup, connection etc.)</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-NxxNMV--1lM/T2upiwDmV0I/AAAAAAAABOg/mfT2y6JNwzg/s320/result_performance_review_1.PNG)](http://4.bp.blogspot.com/-NxxNMV--1lM/T2upiwDmV0I/AAAAAAAABOg/mfT2y6JNwzg/s1600/result_performance_review_1.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - performance review checklist</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://2.bp.blogspot.com/-hkYV7gvnhEc/T2upkH3tNBI/AAAAAAAABOo/DN9visgB8hQ/s320/result_performance_review_2.PNG)](http://2.bp.blogspot.com/-hkYV7gvnhEc/T2upkH3tNBI/AAAAAAAABOo/DN9visgB8hQ/s1600/result_performance_review_2.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - performance review with scoring (by defined rules)</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-qeg9FRpf7RQ/T2uplcY-InI/AAAAAAAABOw/MoKF3Eca26k/s320/result_performance_review_3.PNG)](http://4.bp.blogspot.com/-qeg9FRpf7RQ/T2uplcY-InI/AAAAAAAABOw/MoKF3Eca26k/s1600/result_performance_review_3.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - performance review glossary. Each rule used for performance scoring is explained</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://2.bp.blogspot.com/-FCaEkXRxsM4/T2uphNqBTAI/AAAAAAAABOY/GXJQTcJ9JVA/s320/result_pagespeed.PNG)](http://2.bp.blogspot.com/-FCaEkXRxsM4/T2uphNqBTAI/AAAAAAAABOY/GXJQTcJ9JVA/s1600/result_pagespeed.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - Page Speed optimization check. This one is based on [PageSpeed](http://code.google.com/intl/pl/speed/page-speed/) rules</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-Zzy1glk5U7U/T2upRAWP98I/AAAAAAAABNY/9UqYExymgCc/s320/result_content_breakdown_1.PNG)](http://4.bp.blogspot.com/-Zzy1glk5U7U/T2upRAWP98I/AAAAAAAABNY/9UqYExymgCc/s1600/result_content_breakdown_1.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - content breakdown charts for the first page load</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://3.bp.blogspot.com/-9h-IOXm3rzo/T2upTdnB3MI/AAAAAAAABNg/9Y6Rpum6w04/s320/result_content_breakdown_2.PNG)](http://3.bp.blogspot.com/-9h-IOXm3rzo/T2upTdnB3MI/AAAAAAAABNg/9Y6Rpum6w04/s1600/result_content_breakdown_2.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - content breakdown charts for the next page load</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-7SxdvoNfOT4/T2upVFGgS8I/AAAAAAAABNo/O153zQsoO60/s320/result_content_breakdown_by_domain.PNG)](http://4.bp.blogspot.com/-7SxdvoNfOT4/T2upVFGgS8I/AAAAAAAABNo/O153zQsoO60/s1600/result_content_breakdown_by_domain.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - content breakdown charts with domains specified</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-JTKvDei6PMY/T2upoFcDdzI/AAAAAAAABO8/IdUKyzx02_s/s320/result_screenshots.PNG)](http://4.bp.blogspot.com/-JTKvDei6PMY/T2upoFcDdzI/AAAAAAAABO8/IdUKyzx02_s/s1600/result_screenshots.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - page screen shots for the next page loading steps</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://1.bp.blogspot.com/-XjpsMVyw-hY/T2upcY6F7xI/AAAAAAAABOI/ymQzGcsnyj4/s320/result_grabvideo.PNG)](http://1.bp.blogspot.com/-XjpsMVyw-hY/T2upcY6F7xI/AAAAAAAABOI/ymQzGcsnyj4/s1600/result_grabvideo.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results - page rendering video</td>

</tr>

</tbody>

</table>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">

<tbody>

<tr>

<td style="text-align: center;">[![](http://4.bp.blogspot.com/-nJfwl1WRtX4/T2uppDNNoZI/AAAAAAAABPA/1wUhl5a9KCw/s320/test_history.PNG)](http://4.bp.blogspot.com/-nJfwl1WRtX4/T2uppDNNoZI/AAAAAAAABPA/1wUhl5a9KCw/s1600/test_history.PNG)</td>

</tr>

<tr>

<td class="tr-caption" style="text-align: center;">Test results history. You can compare the results of multiple tests here</td>

</tr>

</tbody>

</table>

Have a nice profiling with WebPageTest!
