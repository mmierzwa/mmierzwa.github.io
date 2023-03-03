---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-8188630730154786454
blogger_orig_url: http://byteloom.blogspot.com/2012/01/error-value-cannot-be-null-parameter.html
date: "2012-01-23T13:58:54Z"
tags:
- sharepoint 2010
title: 'Error: "Value cannot be null. Parameter name: formType" retrieving default
  view page of document library'
---

Few days ago I faced with the problem of linking to default view page of custom document library (for purposes of redirection with Source parameter after uploading new document and filling it's meta data form).

At first I tried using [SPList.Forms](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splist.forms.aspx) collection indexed with [PAGETYPE](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.pagetype.aspx) enumeration as I found in article ["How To Always Link to the Right Application Pages"](http://sharepointsharpener.wordpress.com/2009/05/16/how-to-always-link-to-the-right-application-pages/). But every time I was trying to get the default view object this way:

```csharp
var defaltViewUrl = documentLibrary.Forms[PAGETYPE.PAGE_DEFAULTVIEW].Url;
```

I was getting the following exception:

```
ArgumentNullException: "Value cannot be null. Parameter name: formType"
```

Fortunately this was working fine:

```csharp
var defaultViewUrl = string.Format("{0}/{1}", documentLibrary.ParentWeb.Url, documentLibrary.DefaultView.Url);
```

I've checked that `SPList.Forms` collection returns valid forms/pages objects only for these three values:

* `PAGETYPE.PAGE_DISPLAYFORM`
* `PAGETYPE.PAGE_EDITFORM`
* `PAGETYPE.PAGE_NEWFORM`

For every other it throws arg null exception as described above. This looks definitely like a bad design in Sharepoint API (one of many...) - the collection should be indexed with other enum that contain valid set of values.

You can find some more detailed info about the reason of this strange behavior on the short [StackOverflow thread](http://stackoverflow.com/questions/8837172/error-value-cannot-be-null-parameter-name-formtype-retrieving-default-view) started by my question. How Stefan figured out that error is caused by enum to string conversion? I guess this mystery was revealed with [ILSpy](http://wiki.sharpdevelop.net/ILSpy.ashx) or some similar tool ;-)

Cheers!

{{< blogspot_ref >}}