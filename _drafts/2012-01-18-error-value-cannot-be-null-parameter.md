---
layout: post
title: 'Error: "Value cannot be null. Parameter name: formType" retrieving default view page of document library'
date: '2012-01-18T16:25:00.000+01:00'
author: Marek Mierzwa
tags:
- sharepoint
- document library
- forms
- sharepoint 2010
- PAGETYPE
modified_time: '2012-01-23T13:58:54.638+01:00'
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-8188630730154786454
blogger_orig_url: http://byteloom.blogspot.com/2012/01/error-value-cannot-be-null-parameter.html
---

Few days ago I faced with the problem of linking to default view page of custom document library (for purposes of redirection with Source parameter after uploading new document and filling it's meta data form).<br />At first I tried using <a href="http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splist.forms.aspx" target="_blank">SPList.Forms</a> collection indexed with <a href="http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.pagetype.aspx" target="_blank">PAGETYPE</a> enumeration as I found in article "<a href="http://sharepointsharpener.wordpress.com/2009/05/16/how-to-always-link-to-the-right-application-pages/" target="_blank">How To Always Link to the Right Application Pages</a>". But every time I was trying to get the default view object this way:<br /><br /><pre class="brush: csharp"><br />var defaltViewUrl = documentLibrary.Forms[PAGETYPE.PAGE_DEFAULTVIEW].Url;<br /></pre><br /><br />I was getting the following exception:<br /><br /><pre class="brush: csharp"><br />ArgumentNullException: "Value cannot be null. Parameter name: formType"<br /></pre>Fortunately this was working fine:<br /><br /><pre class="brush: csharp"><br />var defaultViewUrl = string.Format("{0}/{1}", documentLibrary.ParentWeb.Url, documentLibrary.DefaultView.Url);<br /></pre><br /><br />I've checked that SPList.Forms collection returns valid forms/pages objects only for these three values:<br /><ul><li><span class="comment-copy"><span style="font-family: &quot;Courier New&quot;,Courier,monospace;">PAGETYPE.PAGE_DISPLAYFORM</span>,&nbsp;</span></li><li><span class="comment-copy"><span style="font-family: &quot;Courier New&quot;,Courier,monospace;">PAGETYPE.PAGE_EDITFORM</span> and&nbsp;</span></li><li style="font-family: &quot;Courier New&quot;,Courier,monospace;"><span class="comment-copy">PAGETYPE.PAGE_NEWFORM</span></li></ul><span class="comment-copy">For every other it throws arg null exception as described above. This looks definitely like a bad design in Sharepoint API (one of many...) - the collection should be indexed with other enum that contain valid set of values.</span><br /><br /><span class="comment-copy">You can find some more detailed info about the reason of this strange behavior on the short <a href="http://stackoverflow.com/questions/8837172/error-value-cannot-be-null-parameter-name-formtype-retrieving-default-view" target="_blank">stackoverflow thread</a> started by my question. How Stefan figured out that error is caused by enum to string conversion? I guess this mystery was revealed with <a href="http://wiki.sharpdevelop.net/ILSpy.ashx" target="_blank">ILSpy</a> or some similar tool ;-)</span>
