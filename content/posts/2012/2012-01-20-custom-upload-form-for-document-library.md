---
author: Marek Mierzwa
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-6972000166677487449
blogger_orig_url: http://byteloom.blogspot.com/2012/01/custom-upload-form-for-document-library.html
date: "2012-01-20T18:59:00Z"
tags:
- sharepoint 2010
title: Custom upload form for document library in SharePoint 2010 - programmatic approach
---

As I wrote previously the one of my recent projects was to create a new upload page for specific Document Library (based on custom list template) that will use Telerik Upload component (Silverlight) and will entirely replace OOB `upload.aspx` page. Looking for some suggestions how to do this I found few solutions but none of them met my criteria:

1.  Using SharePoint Designer for customizing default upload page - I'm not sure if this could apply to document library; I also rejected this option from start as non-programmatic approach because of problems in future development and maintenance
2.  [New document template redirection trick](http://www.oriolardevol.com/Article/Details/17) - looks simple but this wasn't enough elegant solution for me (I know - who cares, it's SharePoint after all... ;-)
3.  [Creating a custom action in ribbon and hiding the old one with javascript](http://stackoverflow.com/questions/1415638/how-do-i-change-the-upload-page-for-a-particular-document-library-in-sharepoint) - the first part looks quite nice but the second is another hack; also, "Add new document" link at the bottom of default document library view still points to the standard `/_layouts/Upload.aspx` page...
4.  [Creating a custom action in ribbon with custom rendering template for ribbon](http://blogs.msdn.com/b/syedi/archive/2008/07/18/custom-upload-page-in-layouts-for-document-library-and-it-s-navigation-from-upload-menu-in-the-toolbar-bend-it-custom-upload-menu-for-the-document-library.aspx) - very nice solution when you want to change upload pages for all lists on farm, but this is not applicable in my case; still "Add new document" link at the bottom remains

There were also other approaches like changing all related links with jQuery on client side but I would prefer some simple, elegant and server side solution that will not cause any problems on migration to the next version of SharePoint.
<!--more-->

Because document libraries does not have the `New form` (new documents are uploaded or created from Office templates), setting links in list template schema in [Forms](http://msdn.microsoft.com/en-us/library/ie/ms464220.aspx) section **won't work**:

```xml
 <Forms>
  <Form Type="DisplayForm" Url="DispForm.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" />
  <Form Type="EditForm" Url="EditForm.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" />
  <Form Type="NewForm" Url="MyCustomUploadPage.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" />
 </Forms>
```

For the same reason setting [New form template](http://msdn.microsoft.com/en-us/library/ms468901.aspx) in custom content type (inherited from `Document CT`) definition:

```xml
 <FormTemplates xmlns="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">
  <Display>DocumentLibraryForm</Display>
  <Edit>DocumentLibraryForm</Edit>
  <New>DocumentLibraryForm</New>
 </FormTemplates>
```

will not change the upload page. Both methods still works for regular list or CTs not inherited from `Document`.

This may sound obvious for experienced SharePoint developers, but it was not for me. When I understood this I've started looking for methods of changing upload links (marked on picture below) both in ribbon and in default list view.

<figure class="half center">
  <a href="/images/2012/01/custom_library.png" class="image-popup">
	 <img src="/images/2012/01/custom_library.png" alt="Custom Documents Library">
   </a>
	<figcaption>Custom Documents Library</figcaption>
</figure>

The first one - ribbon button - looks fairly simple. I left it for future development after finding some articles describing [creation of Custom Action](http://www.sharepointnutsandbolts.com/2010/01/customizing-ribbon-part-1-creating-tabs.html) and [hiding the existing buttons](http://msdn.microsoft.com/en-us/library/ff408060.aspx). Of course finally I will have to do this.

The second - '+ Add new document' link - was definitely more challenging for me. First I've looked for the CT definition for Document Library and I found Toolbar definition in default view:

```xml
 <Views>
  <View BaseViewID="0" Type="HTML" MobileView="TRUE" TabularView="FALSE" FreeForm="TRUE">
   ...
   <Toolbar Position="After" Type="Freeform">
    <IfHasRights>
     <RightsChoices>
      <RightsGroup PermAddListItems="required" />
     </RightsChoices>
     <Then>
      <Switch>
       <Expr>
        <GetVar Name="MasterVersion" />
       </Expr>
       <Case Value="4"><HTML><![CDATA[<div class="tb"><img src="/_layouts/images/caladd.gif" alt="" /> <a class="ms-addnew" id="idAddNewDoc" href="]]></HTML>
        <HttpVDir /><HTML><![CDATA[/_layouts/Upload.aspx?List=]]></HTML>
        <ListProperty Select="Name" /><HTML><![CDATA[&RootFolder=]]></HTML>
        <GetVar Name="RootFolder" URLEncode="TRUE" /><HTML><![CDATA[" onclick="javascript:NewItem(']]></HTML>
        <ScriptQuote NotAddingQuote="TRUE">
         <HttpVDir />
        </ScriptQuote><HTML><![CDATA[/_layouts/Upload.aspx?List=]]></HTML>
        <ListProperty Select="Name" /><HTML><![CDATA[&RootFolder=]]></HTML>
        <GetVar Name="RootFolder" URLEncode="TRUE" /><HTML><![CDATA[', true);javascript:return false;" target="_self">]]></HTML><HTML>$Resources:core,Add_New_Document;</HTML><HTML><![CDATA[</a></div>]]></HTML>
       </Case>
       <Default><HTML><![CDATA[ <table width="100%" cellpadding="0" cellspacing="0" border="0" > <tr> <td colspan="2" class="ms-partline"><img src="/_layouts/images/blank.gif" width='1' height='1' alt="" /></td> </tr> <tr> <td class="ms-addnew" style="padding-bottom: 3px"> <img src="/_layouts/images/rect.gif" alt="" /> <a class="ms-addnew" id="idAddNewDoc" href="]]></HTML>
        <HttpVDir /><HTML><![CDATA[/_layouts/Upload.aspx?List=]]></HTML>
        <ListProperty Select="Name" /><HTML><![CDATA[&RootFolder=]]></HTML>
        <GetVar Name="RootFolder" URLEncode="TRUE" /><HTML><![CDATA[" onclick="javascript:NewItem(']]></HTML>
        <ScriptQuote NotAddingQuote="TRUE">
         <HttpVDir />
        </ScriptQuote><HTML><![CDATA[/_layouts/Upload.aspx?List=]]></HTML>
        <ListProperty Select="Name" /><HTML><![CDATA[&RootFolder=]]></HTML>
        <GetVar Name="RootFolder" URLEncode="TRUE" /><HTML><![CDATA[', true);javascript:return false;" target="_self">]]></HTML><HTML>$Resources:core,Add_New_Document;</HTML><HTML><![CDATA[</a> </td> </tr> <tr><td><img src="/_layouts/images/blank.gif" width='1' height='5' alt="" /></td></tr> </table>]]></HTML>
       </Default>
      </Switch>
     </Then>
    </IfHasRights>
   </Toolbar>
   ...
  </View>
  ...
 </Views>
```

which was promising but I've noticed that 'Add new document' in the view that is actually displayed is different than the one defined above (it calls `NewItem2()` JS function instead of `NewItem()`).

After next few hours of searching and digging in 14 SharePoint folder I finally found the answer why the link at the bottom of Document Library default view is rendered in such way. In SharePoint 2010 every List View is actually `XsltListViewWebPart` which is well described in [MSDN](http://msdn.microsoft.com/en-us/library/ff604021.aspx) (this was also something new for me as for SP rookie). As the name suggest rendering is based on (quite complex) XSL transformations that for default views are defined in `vwstyles.xsl` imported in `main.xsl` which is linked in the following line in View definition of list template:

```xml
 <XslLink Default="TRUE">main.xsl</XslLink>
```

In case of 'Add new document' link transformations end on this template:

```xml
 <xsl:template name="Freeform">
  ...
  <xsl:variable name="Url">
   <xsl:choose>
    <xsl:when test="List/@TemplateType='119'"><xsl:value-of select="$HttpVDir"/>/_layouts/CreateWebPage.aspx?List=<xsl:value-of select="$List"/>&RootFolder=<xsl:value-of select="$XmlDefinition/List/@RootFolder"/></xsl:when>
    <xsl:when test="$IsDocLib"><xsl:value-of select="$HttpVDir"/>/_layouts/Upload.aspx?List=<xsl:value-of select="$List"/>&RootFolder=<xsl:value-of select="$XmlDefinition/List/@RootFolder"/></xsl:when>
    <xsl:otherwise><xsl:value-of select="$ENCODED_FORM_NEW"/></xsl:otherwise>
   </xsl:choose>
  </xsl:variable>
  ...
 </xsl:template>
```

which renders the link to default `Upload.aspx` page in `_layouts` folder. As you see for Document Libraries this link is always the same, no matter what you will put in New form in CT definition or list template.

The simple, programmatic and server side solution for changing the link I was looking for and which worked for me was to deploy my own xsl (i.e. `custom_views.xsl`):

```xml
 <xsl:stylesheet xmlns:x="http://www.w3.org/2001/XMLSchema" xmlns:d="http://schemas.microsoft.com/sharepoint/dsp" version="1.0"
     exclude-result-prefixes="xsl msxsl ddwrt" xmlns:ddwrt="http://schemas.microsoft.com/WebParts/v2/DataView/runtime"
     xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:__designer="http://schemas.microsoft.com/WebParts/v2/DataView/designer"
     xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:sppchemas-microsoft-com:xslt"
     xmlns:SharePoint="Microsoft.SharePoint.WebControls" xmlns:ddwrt2="urn:frontpage:internal" ddwrt:oob="true">

  <xsl:include href="/_layouts/xsl/main.xsl"/>
  <xsl:include href="/_layouts/xsl/internal.xsl"/>

  <xsl:template name="Freeform">
   <xsl:param name="AddNewText"/>
   <xsl:param name="ID"/>
   <xsl:variable name="Url">
    <xsl:choose>
     <xsl:when test="List/@TemplateType='119'">
      <xsl:value-of select="$HttpVDir"/>/_layouts/CreateWebPage.aspx?List=<xsl:value-of select="$List"/>&RootFolder=<xsl:value-of select="$XmlDefinition/List/@RootFolder"/>
     </xsl:when>
     <xsl:when test="$IsDocLib">
      <xsl:value-of select="$HttpVDir"/>/_layouts/LargeFileUploadWithSLToSP/LibraryUpload.aspx?documentLibraryId=<xsl:value-of select="$List"/>&RootFolder=<xsl:value-of select="$XmlDefinition/List/@RootFolder"/>
     </xsl:when>
     <xsl:otherwise>
      <xsl:value-of select="$ENCODED_FORM_NEW"/>
     </xsl:otherwise>
    </xsl:choose>
   </xsl:variable>
   ...
  </xsl:template>
 </xsl:stylesheet>
```

in a mapped VS solution folder (which will overwrite only the "Freeform" template) to Layouts and put the link to it in `XslLink` in `View` definition:

```xml
 <View BaseViewID="1" Type="HTML" WebPartZoneID="Main" DisplayName="$Resources:core,All_Documents;" DefaultView="TRUE" DefaultViewForContentType="TRUE" MobileView="True" MobileDefaultView="True" SetupPath="pages\viewpage.aspx" ImageUrl="/_layouts/images/dlicon.png" Url="Forms/AllItems.aspx">
  <XslLink Default="true">custom_views.xsl</XslLink>
  ...
 </View>
```

Here is a sample Visual Studio 2010 solution structure:

<figure class="half center">
  <a href="/images/2012/01/vs_solution_for_upload.png" class="image-popup">
	 <img src="/images/2012/01/vs_solution_for_upload.png" alt="VS Solution">
   </a>
	<figcaption>Visual Studio solution structure</figcaption>
</figure>

The results (with Telerik Silverlight upload component) are shown below:

<figure class="half center">
  <a href="/images/2012/01/custom_upload_1.png" class="image-popup">
	 <img src="/images/2012/01/custom_upload_1.png" alt="Custom upload solution (1)">
   </a>
	<figcaption>Custom upload solution (1)</figcaption>
</figure>

<figure class="half center">
  <a href="/images/2012/01/custom_upload_2.png" class="image-popup">
	 <img src="/images/2012/01/custom_upload_1.png" alt="Custom upload solution (2)">
   </a>
	<figcaption>Custom upload solution (2)</figcaption>
</figure>

This satisfies all my requirements: it's a full programmatic server side solution, it does not require any javascript tricks (high risk that in next version of SP will not work, when for example the css style names will change), it can be applied selectively to specific Document Libraries based on custom list templates (no need to change the default behavior on entire WFE) and it is simple.

The VS solution with code samples for this article is available on [GitHub](https://github.com/mmierzwa/byteloom-codesamples/tree/master/LargeFileUploadWithSLToSP)

I hope this short article will spare your time.

Happy SharePointing!
