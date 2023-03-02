---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-7343601350174608200
blogger_orig_url: http://byteloom.blogspot.com/2012/09/sharepoint-taxonomies-struggling-with.html
date: "2012-09-28T09:47:00Z"
tags:
- sharepoint 2010
title: SharePoint Taxonomies - Struggling with duplicated default labels
---

If you have ever had to load large number of data into SharePoint MMD service or build taxonomies automatically you have likely encountered the following problem:  

```
Microsoft.SharePoint.Taxonomy.TermStoreOperationException:
  There is already a term with the same default label and parent term.  
```



<pre class="brush: text">at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.b__2b()  
at Microsoft.Office.Server.Security.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)  
at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.b__2a()  
at Microsoft.Office.Server.Utilities.MonitoredScopeWrapper.RunWithMonitoredScope(Action code)  
at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun, Double operationTimeoutFactor)  
at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun)  
at Microsoft.SharePoint.Taxonomy.Internal.TaxonomyProxyAccess.Write(String data)  
at Microsoft.SharePoint.Taxonomy.Internal.DataAccessManager.Write(String data)  
at Microsoft.SharePoint.Taxonomy.Internal.Sandbox.CommitSandbox()  
at Microsoft.SharePoint.Taxonomy.TermStore.CommitAll()  
</pre>

Not very informative stack trace as you see... First, this happens during the commit, not on creation of new term. Second, there is no information about the conflicting labels. (This is because the validation is done in SQL after building the whole changed structure, which you can check in ULS logs). This error could be quite frustrating, which I've experienced myself (I lost three days on this when I was implementing full synchronization of large tree structure with SP taxonomies). I will try to give some hints all of those which are struggling with duplicated labels.  

## Remember about default label conversions

As I wrote in my previous post - ["SharePoint Taxonomies - Labels with forbidden characters"]({{ site.baseurl }}{% post_url 2012/2012-09-20-sharepoint-taxonomies-labels-with %}), SharePoint performs some [transformations on label strings](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.taxonomy.term.name.aspx): trim, replacing consecutive spaces into one and ampersand with its wider Unicode equivalent (\uFF06).  

The result of above is that label `" black duck & goose "` and `"black duck ＆ goose"` will be treated as the same label string.  

If you decide to replace the characters forbidden in labels (`;"<>|&tab`) with some others you will have to keep in mind these transformation to.  

## Labels are compared case invariant

This means that label `"BlAcK DuCK"` will be treated as the same string as label `"black duck"`.  

## Label changed after creation before commit is not really changed...

This was my case (3 days of digging...). Consider the following code:  

```csharp
//parent (TermSetItem) and termStore is defined before this snippet  

//...  

var term1Label = "black duck";  
var term1 = parent.CreateTerm(term1Label, 1033);  

var term2Label = "black duck";  
var term2 = parent.CreateTerm(term2Label, 1033);  

term2.Name = "white goose";  

//...  

termStore.CommitAll();  
```

If you think this code will run without any problem you are wrong. As I did.  
You must ensure the label uniqueness between sibling nodes in the time when they are created. Of course this fact is not documented in API on MSDN...  

As some used to say - there is a thin line between bug and feature ;-)

## How to hunt the vampire down?

The last tip I can give you is a little piece of code you can run just before the commit in order to find the problematic labels:  

```csharp
//this will give you all terms in the group (you can narrow the scope, i.e. to particular term set, if you like  
var terms = termGroup.TermSets.SelectMany(termSet => termSet.GetAllTerms());  

//this will actually return the terms with duplicated labels (for default LCID) grouped by parent-label pair  
var duplicates = terms.  
    GroupBy(term => string.Format("{0}:{1}", getParentGuid(term), term.Name.ToLowerInvariant())).  
    Where(group => group.Count() > 1).  
    SelectMany(group => group).  
    ToList();  

//now you can do with them whatever you like, i.e. write them out to the file:  
var termsStr = new StringBuilder();  
termStr.Append("Term GUID;Term default label;Parent GUID");  
foreach (var term in duplicates)  
{  
    termsStr.AppendFormat("{0};\"{1}\";{2}\n", term.Id, term.Name, getParentGuid(term));  
}  
File.AppendAllText(@"c:\duplicatedTerms.txt", termsStr.ToString(), new UTF32Encoding());  
```

```csharp
string getParentGuid(Term term)  
{  
    return term.Parent != null ? term.Parent.Id.ToString() : term.TermSet.Id.ToString();  
}  
```

Unfortunately this won't help if you change the label after creation like mentioned above.  

I hope this little guideline will save you some time.

Happy SharePointing!

{{< blogspot_ref >}}