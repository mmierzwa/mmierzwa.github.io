---
layout: post
title: Replace attachment in document library without changing version number
date: 2012-03-06 01:03:00 +0100
tags: [sharepoint 2010]
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-2358495401711229210
blogger_orig_url: http://byteloom.blogspot.com/2012/03/replace-attachment-in-document-library.html
---

Lately I was looking for some example on how to replace the attachment in document library (SP2010) without changing the version number but without any results. If you've faced the same problem here is the solution<!-- more -->:

```csharp
SPWeb webSite = SPContext.Current.Web;  
SPDocumentLibrary library = webSite.Lists[libraryName] as SPDocumentLibrary;  

if (library == null)  
{  
    throw new SPException("There is no document library named " + libraryName);  
}  

using (MemoryStream docStream = new MemoryStream())  
{  
    BinaryWriter docWriter = new BinaryWriter(docStream);  
    docWriter.Write(yourContentString);  

    SPFile file = find(library, fileName);  

    if (file == null)  
    {  
        throw new SPException("File " + fileName + " not found");  
    }  

    file.CheckOut();  
    file.SaveBinary(docStream);  
    file.CheckIn(string.Empty, SPCheckinType.OverwriteCheckIn);  

    docWriter.Close();  
}  
```

Utility method `find()` - find a file by name - is defined as follow:

```csharp
private static SPFile find(SPDocumentLibrary library, string fileName)  
{  
    SPFileCollection files = library.RootFolder.Files;  
    for (int fileIdx = 0; fileIdx < files.Count; fileIdx++)  
    {  
        if (files[fileIdx].Name == fileName)  
        {  
            return files[fileIdx];  
        }  
    }  

    return null;  
}  
```

Im assuming that the file has already been uploaded to document library. The most important parts are:

1.  Retrieve reference to the file ([SPFile](http://msdn.microsoft.com/en-us/library/ms461145.aspx)). I've done this by iterating through all files in root folder of document library, but actually you can do this however you want
2.  Check out the file ([SPFile.CheckOut()](http://msdn.microsoft.com/en-us/library/ms454425.aspx)):
```csharp
file.CheckOut();
```
3.  Replace the attachment. Convenient method for doing this is [SPFile.SaveBinary](http://msdn.microsoft.com/en-us/library/ms465421.aspx) which takes stream with content as an argument (I've used `MemoryStream` to read the content from a text box in my PoC, but you can pass the FileStream or something else):
```csharp
file.SaveBinary(docStream);
```
4.  Check-in the file ([SPFile.CheckIn()](http://msdn.microsoft.com/en-us/library/ms412209.aspx)). In order to save the attachment without increasing the version number you must specify [SPCheckinType](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spcheckintype.aspx) as `OverwriteCheckIn`
```csharp
    file.CheckIn(string.Empty, SPCheckinType.OverwriteCheckIn);  
```

The alternative method is by using [SPListItem.SystemUpdate](http://msdn.microsoft.com/en-us/library/ms481195.aspx) with `incrementListItemVersion` set `false`. `SystemUpdate` will not change item modification date and modified by fields values. In this approach you will work with document library item instead of file.  
Additionally if you want to change the attachment name, you can do this by with [SPFile.MoveTo](http://msdn.microsoft.com/en-us/library/ms438892.aspx) method (after changing the original attachment content):

```csharp
if (!string.IsNullOrEmpty(newFileName))  
{  
    string newPath = string.Format("{0}/{1}", file.ParentFolder.Url, newFileName);  
    file.MoveTo(newPath, true);  
}  
```

The VS solution with code samples for this article is available on [GitHub](https://github.com/mmierzwa/byteloom-codesamples/tree/master/ReplaceAttachement/)

Happy SharePointing!
