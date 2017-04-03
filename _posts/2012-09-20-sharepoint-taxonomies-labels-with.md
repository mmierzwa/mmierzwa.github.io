---
layout: post
title: SharePoint Taxonomies - Labels with forbiden characters
date: 2012-09-20 17:02:00 +0200
tags: [sharepoint 2010]
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-442845803444553887
blogger_orig_url: http://byteloom.blogspot.com/2012/09/sharepoint-taxonomies-labels-with.html
---

Recently, when I was working on mechanism of automatic synchronization of tree structures provided by web service to SharePoint taxonomies, I came across an error like this:  

```
The value '(<= 0320-775, 0550-5/7)' is invalid. It probably contains invalid characters or is too long.  
Parameter name: name  
```

with the following stacktrace fragment:  

```
at Microsoft.SharePoint.Taxonomy.Internal.CommonValidator.ValidateName(String name, String parameterName)  
at Microsoft.SharePoint.Taxonomy.TermSetItem.CreateTerm(String name, Int32 lcid, Guid newTermId)  
...  
```
<!-- more -->

Looking for some help in SharePoint API documentation for method [TermSetItem.CreateTerm()](http://msdn.microsoft.com/en-us/library/ee583797.aspx) I found the explanation for this error:  

> The labelName value will be normailized to trim consecutive spaces into one and replace the & character with the wide character version of the character (\\uFF06). It must be non-empty, cannot exceed 255 characters, and cannot contain any of the following characters ;"< >|&tab

Yeah, this is SharePoint... No surprise, I would say ;)  

Since I had to put the forbidden characters into the term labels somehow, my first thought was to encode them as HTML entities. But as you may noticed it is impossible - the ampersand character is replaced by similar looking unicode character with code `\uFF06`.  

My second try - why not to try the same as MS SharePoint developers? You will find many UTF-16 characters that look almost exactly the same as the characters from ASCII set. The replacements for the forbidden characters in UTF-8 are:  

Original ASCII char | UTF-8 replacement code | UTF-8 char
:-------------------|:-----------------------|:-----------
\                   | \\uFF5C                | ｜
"                   | \\uFF02                | ＂         
<                   | \\uFF1C                | ＜         
\>                  | \\uFF1E                | ＞         

Unfortunetly I found no replacement for semicolon character so I'm replacing it with coma. You can find more information about these characters on [FileFormat](http://www.fileformat.info/info/unicode).  

Of course you will have to create the term grammatically to put them into term label (or copy them from some unicode table, such as [FileFormat](http://www.fileformat.info/info/unicode)). Here is a utility method that should "normalize" the strings for `CreateTerm()` method:  

```csharp
public static string ReplaceIllegalCharacters(string termLabel)  
{  
    return termLabel.  
        Replace("\t", " ").  
        Replace(";", ",").  
        Replace("\"", "\uFF02").  
        Replace("<", "\uFF1C").  
        Replace(">", "\uFF1E").  
        Replace("|", "\uFF5C");  
}  
```

This works fine either in web interface and other clients (I've tested it with MS Word 2010 and Lotus Notes widget for SharePoint integration - [Harmon.IE](http://harmon.ie/)). ~~I will put here some screenshots soon.~~

<figure class="half center">
  <a href="/images/2012/09/sp_mmd_picker.png" class="image-popup">
	 <img src="/images/2012/09/sp_mmd_picker.png" alt="SharePoint web UI term picker">
   </a>
	<figcaption>SharePoint web UI term picker</figcaption>
</figure>

<figure class="half center">
  <a href="/images/2012/09/word_mmd_picker.png" class="image-popup">
	 <img src="/images/2012/09/word_mmd_picker.png" alt="MS Word term picker">
   </a>
	<figcaption>MS Word term picker</figcaption>
</figure>

<figure class="half center">
  <a href="/images/2012/09/harmonie_mmd_picker.png" class="image-popup">
	 <img src="/images/2012/09/harmonie_mmd_picker.png" alt="Harmon.IE term picker">
   </a>
	<figcaption>Harmon.IE term picker</figcaption>
</figure>

<figure class="half center">
  <a href="/images/2012/09/term_store_management_tool.png" class="image-popup">
	 <img src="/images/2012/09/term_store_management_tool.png" alt="SharePoint TermStore management tool">
   </a>
	<figcaption>SharePoint TermStore management tool (Central Administration)</figcaption>
</figure>

Happy SharePointing!
