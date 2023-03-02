---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-1307989842451191515
blogger_orig_url: http://byteloom.blogspot.com/2012/07/custom-xmlresolver-for-embeded-dtd.html
date: "2012-07-03T08:28:00Z"
tags:
- .net
- xml
title: Custom XmlResolver for embeded DTD
---

Writing a component for parsing XML files with `XMLSerializer` I had to provide DTD validation (DTD file was already created long time ago so there was no sense for creating XSD schema). The component must have been able to work in console application and web app (as a SharePoint timer job) so there was no chance to guarantee the same paths for DTD file (which was always specified in doctype directive in processed files). In such situation I've decided to deliver the DTD file as embedded resource in component assembly.  
<!--more-->
If you had ever faced the similar problem you probably know that standard `XmlResolver` implementations in .Net framework will not able to find DTD in such location. If you try you will probably get an exception like this:  

```
System.InvalidOperationException: There is an error in XML document (0, 0).
---> System.Xml.XmlException: Cannot resolve external DTD subset - public ID = '', system ID = 'yourDTDFileDefinedInDoctype.dtd'.  
```

or something similar. Thats why you have to provide the XmlSerializer some your own mechanism that will find the DTD file - custom [XmlResolver](http://msdn.microsoft.com/en-us/library/system.xml.xmlresolver.aspx).  
There is a good example how to do this on [MSDN](http://msdn.microsoft.com/en-us/library/bb669135.aspx). I've changed it a little bit to cover the use case with embedded resources.  

```csharp
public class XmlEmbededResolver : XmlUrlResolver  
{  
    private readonly string dtdResourcePathPrefix;  

    public XmlEmbededResolver(string dtdResourcePathPrefix)  
    {  
        this.dtdResourcePathPrefix = dtdResourcePathPrefix;  
    }  

    public override object GetEntity(Uri absoluteUri, string role, Type ofObjectToReturn)  
    {  
        if (absoluteUri == null)  
        {  
            throw new ArgumentNullException("absoluteUri");  
        }  

        if (isDocumentTypeDefinitionFile(absoluteUri) &&  
            (ofObjectToReturn == null || ofObjectToReturn == typeof(Stream)))  
        {  
            var filePath = dtdResourcePathPrefix + getFileName(absoluteUri);  
            var resourceStream = Assembly.  
                GetExecutingAssembly().  
                GetManifestResourceStream(filePath);  

            if (resourceStream == null)  
            {  
                throw new FileNotFoundException("Embeded DTD file not found", filePath);  
            }  

            return resourceStream;  
        }  

        return base.GetEntity(absoluteUri, role, ofObjectToReturn);  
    }
}
```

Parameter `dtdResourcePathPrefix` in constructor points the location of DTD (without file name) in the assembly (usually the default namespace + folder path).  
The utility method `isDocumentTypeDefinitionFile()` is used for determining if entity passed by `XmlSerializer` to resolve is actually an DTD file (`XmlResolver` is used resolve any kind of external resources defined in XML file):  

```csharp
private static bool isDocumentTypeDefinitionFile(Uri absoluteUri)  
{  
    return absoluteUri.IsFile &&  
    absoluteUri.AbsolutePath.EndsWith(".dtd");  
}  
```

`getFileName()`, as its name suggests, returns the name of DTD:  

```csharp
private static string getFileName(Uri absoluteUri)  
{  
    return absoluteUri.Segments.Last();  
}  
```

Using the custom resolver is pretty simple - you just pass it as setting to `XmlReader` in `Create()` method:  

```csharp
var readerSettings = new XmlReaderSettings  
{  
    ProhibitDtd = false,  
    ValidationType = ValidationType.DTD,  
    XmlResolver = new XmlEmbededResolver(dtdResourcePathPrefix)  
};  

using (var reader = XmlReader.Create(xmlFileStream, readerSettings))  
{  
    var serializer = new XmlSerializer(typeof (YourXmlDataType));  
    var rawImportEntry = (YourXmlDataType)serializer.Deserialize(reader);  
    // ...  
}  
```

I can easily imagine other implementations of `XmlResolver` - for example to provide external resources from the web services (of course if someone see any sense in creating such thing ;-)

Happy coding!

{{< blogspot_ref >}}