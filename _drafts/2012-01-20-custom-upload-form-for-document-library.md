---
layout: post
title: Custom upload form for document library in SharePoint 2010 - programmatic approach
date: '2012-01-20T18:59:00.001+01:00'
author: Marek Mierzwa
tags:
- sharepoint
- document library
- upload
- XsltListViewWebPart
- telerik
- view
- forms
- list template
- sharepoint 2010
- xsl
modified_time: '2012-03-06T08:58:12.595+01:00'
thumbnail: http://1.bp.blogspot.com/-IIerPCLFLBo/Txml-3wiHRI/AAAAAAAABKc/ZhvQj8IOdTg/s72-c/custom_library.PNG
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-6972000166677487449
blogger_orig_url: http://byteloom.blogspot.com/2012/01/custom-upload-form-for-document-library.html
---

<div style="font-family: inherit;"><span style="font-size: small;">As I wrote previously the one of my recent projects was to create a new upload page for specific Document Library (based on custom list template) that will use Telerik Upload component (Silverlight) and will entirely replace OOB upload.aspx page. Looking for some suggestions how to do this I found few solutions but none of them met my criteria:</span></div><ol style="font-family: inherit;"><li><span style="font-size: small;">Using SharePoint Designer for customizing default upload page - I'm not sure if this could apply to document library;  I also rejected this option from start as non-programmatic approach because of problems in future development and maintenance</span></li><li><span style="font-size: small;"><a href="http://www.oriolardevol.com/Article/Details/17" target="_blank">New document template redirection trick</a> - looks simple but this wasn't enough elegant solution for me (I know - who cares, it's SharePoint after all... ;-)</span></li><li><span style="font-size: small;"><a href="http://stackoverflow.com/questions/1415638/how-do-i-change-the-upload-page-for-a-particular-document-library-in-sharepoint" target="_blank">Creating a custom action in ribbon and hiding the old one with javascript</a> - the first part looks quite nice but the second is another hack; also, "Add new document" link at the bottom of default document library view still points to the standard /_layouts/Upload.aspx page...</span></li><li><span style="font-size: small;"><a href="http://blogs.msdn.com/b/syedi/archive/2008/07/18/custom-upload-page-in-layouts-for-document-library-and-it-s-navigation-from-upload-menu-in-the-toolbar-bend-it-custom-upload-menu-for-the-document-library.aspx" target="_blank">Creating a custom action in ribbon with custom rendering template for ribbon</a> - very nice solution when you want to change upload pages for all lists on farm, but this is not applicable in my case; still "Add new document" link at the bottom remains</span></li></ol><div style="font-family: inherit;"><span style="font-size: small;">There were also other approaches like changing all related links with jQuery on client side but I would prefer some simple, elegant and server side solution that will not cause any problems on migration to the next version of SharePoint.</span></div><a name='more'></a> <div style="font-family: inherit;"><span style="font-size: small;">Because document libraries does not have the New form (new documents are uploaded or created from Office templates), setting links in list template schema in <a href="http://msdn.microsoft.com/en-us/library/ie/ms464220.aspx" target="_blank">Forms</a> section <b>won't work</b>:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;Forms><br />  &lt;Form Type="DisplayForm" Url="DispForm.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" /><br />  &lt;Form Type="EditForm" Url="EditForm.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" /><br />  &lt;Form Type="NewForm" Url="MyCustomUploadPage.aspx" SetupPath="pages\form.aspx" WebPartZoneID="Main" /><br /> &lt;/Forms><br /></pre><div style="font-family: inherit;"><span style="font-size: small;">For the same reason setting New <a href="http://msdn.microsoft.com/en-us/library/ms468901.aspx" target="_blank">form template</a> in custom content type (inherited from Document CT) definition:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;FormTemplates xmlns="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"><br />  &lt;Display>DocumentLibraryForm&lt;/Display><br />  &lt;Edit>DocumentLibraryForm&lt;/Edit><br />  &lt;New>DocumentLibraryForm&lt;/New><br /> &lt;/FormTemplates><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">will not change the upload page. Both methods still works for regular list or CTs not inherited from Document.</span></div><div style="font-family: inherit;"><span style="font-size: small;">This may sound obvious for experienced SharePoint developers, but it was not for me. When I understood this I've started looking for methods of changing upload links (marked on picture below) both in ribbon and in default list view.</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div class="separator" style="clear: both; font-family: inherit; text-align: center;"><span style="font-size: small;"><a href="http://1.bp.blogspot.com/-IIerPCLFLBo/Txml-3wiHRI/AAAAAAAABKc/ZhvQj8IOdTg/s1600/custom_library.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="211" src="http://1.bp.blogspot.com/-IIerPCLFLBo/Txml-3wiHRI/AAAAAAAABKc/ZhvQj8IOdTg/s400/custom_library.PNG" width="400" /></a></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">The first one - ribbon button - looks fairly simple. I left it for future development after finding some articles describing <a href="http://www.sharepointnutsandbolts.com/2010/01/customizing-ribbon-part-1-creating-tabs.html" target="_blank">creation of Custom Action</a> and <a href="http://msdn.microsoft.com/en-us/library/ff408060.aspx" target="_blank">hiding the existing buttons</a>. Of course finally I will have to do this.</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">The second - '+ Add new document' link - was definitely more challenging for me. First I've looked for the CT definition for Document Library and I found Toolbar definition in default view:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;Views><br />  &lt;View BaseViewID="0" Type="HTML" MobileView="TRUE" TabularView="FALSE" FreeForm="TRUE"><br />   ...<br />   &lt;Toolbar Position="After" Type="Freeform"><br />    &lt;IfHasRights><br />     &lt;RightsChoices><br />      &lt;RightsGroup PermAddListItems="required" /><br />     &lt;/RightsChoices><br />     &lt;Then><br />      &lt;Switch><br />       &lt;Expr><br />        &lt;GetVar Name="MasterVersion" /><br />       &lt;/Expr><br />       &lt;Case Value="4">&lt;HTML>&lt;![CDATA[&lt;div class="tb">&lt;img src="/_layouts/images/caladd.gif" alt="" />&#160;&lt;a class="ms-addnew" id="idAddNewDoc" href="]]>&lt;/HTML><br />        &lt;HttpVDir />&lt;HTML>&lt;![CDATA[/_layouts/Upload.aspx?List=]]>&lt;/HTML><br />        &lt;ListProperty Select="Name" />&lt;HTML>&lt;![CDATA[&RootFolder=]]>&lt;/HTML><br />        &lt;GetVar Name="RootFolder" URLEncode="TRUE" />&lt;HTML>&lt;![CDATA[" onclick="javascript:NewItem(']]>&lt;/HTML><br />        &lt;ScriptQuote NotAddingQuote="TRUE"><br />         &lt;HttpVDir /><br />        &lt;/ScriptQuote>&lt;HTML>&lt;![CDATA[/_layouts/Upload.aspx?List=]]>&lt;/HTML><br />        &lt;ListProperty Select="Name" />&lt;HTML>&lt;![CDATA[&RootFolder=]]>&lt;/HTML><br />        &lt;GetVar Name="RootFolder" URLEncode="TRUE" />&lt;HTML>&lt;![CDATA[', true);javascript:return false;" target="_self">]]>&lt;/HTML>&lt;HTML>$Resources:core,Add_New_Document;&lt;/HTML>&lt;HTML>&lt;![CDATA[&lt;/a>&lt;/div>]]>&lt;/HTML><br />       &lt;/Case><br />       &lt;Default>&lt;HTML>&lt;![CDATA[ &lt;table width="100%" cellpadding="0" cellspacing="0" border="0" > &lt;tr> &lt;td colspan="2" class="ms-partline">&lt;img src="/_layouts/images/blank.gif" width='1' height='1' alt="" />&lt;/td> &lt;/tr> &lt;tr> &lt;td class="ms-addnew" style="padding-bottom: 3px"> &lt;img src="/_layouts/images/rect.gif" alt="" />&#160;&lt;a class="ms-addnew" id="idAddNewDoc" href="]]>&lt;/HTML><br />        &lt;HttpVDir />&lt;HTML>&lt;![CDATA[/_layouts/Upload.aspx?List=]]>&lt;/HTML><br />        &lt;ListProperty Select="Name" />&lt;HTML>&lt;![CDATA[&RootFolder=]]>&lt;/HTML><br />        &lt;GetVar Name="RootFolder" URLEncode="TRUE" />&lt;HTML>&lt;![CDATA[" onclick="javascript:NewItem(']]>&lt;/HTML><br />        &lt;ScriptQuote NotAddingQuote="TRUE"><br />         &lt;HttpVDir /><br />        &lt;/ScriptQuote>&lt;HTML>&lt;![CDATA[/_layouts/Upload.aspx?List=]]>&lt;/HTML><br />        &lt;ListProperty Select="Name" />&lt;HTML>&lt;![CDATA[&RootFolder=]]>&lt;/HTML><br />        &lt;GetVar Name="RootFolder" URLEncode="TRUE" />&lt;HTML>&lt;![CDATA[', true);javascript:return false;" target="_self">]]>&lt;/HTML>&lt;HTML>$Resources:core,Add_New_Document;&lt;/HTML>&lt;HTML>&lt;![CDATA[&lt;/a> &lt;/td> &lt;/tr> &lt;tr>&lt;td>&lt;img src="/_layouts/images/blank.gif" width='1' height='5' alt="" />&lt;/td>&lt;/tr> &lt;/table>]]>&lt;/HTML><br />       &lt;/Default><br />      &lt;/Switch><br />     &lt;/Then><br />    &lt;/IfHasRights><br />   &lt;/Toolbar><br />   ...<br />  &lt;/View><br />  ...<br /> &lt;/Views><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">which was promising but I've noticed that 'Add new document' in the view that is actually displayed is different than the one defined above (it calls NewItem2() JS function instead of NewItem()).</span></div><div style="font-family: inherit;"><span style="font-size: small;">After next few hours of searching and digging in 14 SharePoint folder I finally found the answer why the link at the bottom of Document Library default view is rendered in such way. In SharePoint 2010 every List View is actually XsltListViewWebPart which is well described in <a href="http://msdn.microsoft.com/en-us/library/ff604021.aspx" target="_blank">MSDN</a> (this was also something new for me as for SP rookie). As the name suggest rendering is based on (quite complex) XSL transformations that for default views are defined in vwstyles.xsl imported in main.xsl which is linked in the following line in View definition of list template:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;XslLink Default="TRUE">main.xsl&lt;/XslLink><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">In case of 'Add new document' link transformations end on this template:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;xsl:template name="Freeform"><br />  ...<br />  &lt;xsl:variable name="Url"><br />   &lt;xsl:choose><br />    &lt;xsl:when test="List/@TemplateType='119'">&lt;xsl:value-of select="$HttpVDir"/>/_layouts/CreateWebPage.aspx?List=&lt;xsl:value-of select="$List"/>&amp;RootFolder=&lt;xsl:value-of select="$XmlDefinition/List/@RootFolder"/>&lt;/xsl:when><br />    &lt;xsl:when test="$IsDocLib">&lt;xsl:value-of select="$HttpVDir"/>/_layouts/Upload.aspx?List=&lt;xsl:value-of select="$List"/>&amp;RootFolder=&lt;xsl:value-of select="$XmlDefinition/List/@RootFolder"/>&lt;/xsl:when><br />    &lt;xsl:otherwise>&lt;xsl:value-of select="$ENCODED_FORM_NEW"/>&lt;/xsl:otherwise><br />   &lt;/xsl:choose><br />  &lt;/xsl:variable><br />  ...<br /> &lt;/xsl:template><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">which renders the link to default Upload.aspx page in _layouts folder. As you see for Document Libraries this link is always the same, no matter what you will put in New form in CT definition or list template.</span></div><div style="font-family: inherit;"><span style="font-size: small;">The simple, programmatic and server side solution for changing the link I was looking for and which worked for me was to deploy my own xsl (i.e. custom_views.xsl):</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;xsl:stylesheet xmlns:x="http://www.w3.org/2001/XMLSchema" xmlns:d="http://schemas.microsoft.com/sharepoint/dsp" version="1.0"<br />     exclude-result-prefixes="xsl msxsl ddwrt" xmlns:ddwrt="http://schemas.microsoft.com/WebParts/v2/DataView/runtime"<br />     xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:__designer="http://schemas.microsoft.com/WebParts/v2/DataView/designer"<br />     xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:sppchemas-microsoft-com:xslt"<br />     xmlns:SharePoint="Microsoft.SharePoint.WebControls" xmlns:ddwrt2="urn:frontpage:internal" ddwrt:oob="true"><br /><br />  &lt;xsl:include href="/_layouts/xsl/main.xsl"/><br />  &lt;xsl:include href="/_layouts/xsl/internal.xsl"/><br /><br />  &lt;xsl:template name="Freeform"><br />   &lt;xsl:param name="AddNewText"/><br />   &lt;xsl:param name="ID"/><br />   &lt;xsl:variable name="Url"><br />    &lt;xsl:choose><br />     &lt;xsl:when test="List/@TemplateType='119'"><br />      &lt;xsl:value-of select="$HttpVDir"/>/_layouts/CreateWebPage.aspx?List=&lt;xsl:value-of select="$List"/>&amp;RootFolder=&lt;xsl:value-of select="$XmlDefinition/List/@RootFolder"/><br />     &lt;/xsl:when><br />     &lt;xsl:when test="$IsDocLib"><br />      &lt;xsl:value-of select="$HttpVDir"/>/_layouts/LargeFileUploadWithSLToSP/LibraryUpload.aspx?documentLibraryId=&lt;xsl:value-of select="$List"/>&amp;RootFolder=&lt;xsl:value-of select="$XmlDefinition/List/@RootFolder"/><br />     &lt;/xsl:when><br />     &lt;xsl:otherwise><br />      &lt;xsl:value-of select="$ENCODED_FORM_NEW"/><br />     &lt;/xsl:otherwise><br />    &lt;/xsl:choose><br />   &lt;/xsl:variable><br />   ...<br />  &lt;/xsl:template><br /> &lt;/xsl:stylesheet><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">in a mapped VS solution folder (which will overwrite only the "Freeform" template) to Layouts and put the link to it in XslLink in View definition:</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><pre class="brush: xml"><br /> &lt;View BaseViewID="1" Type="HTML" WebPartZoneID="Main" DisplayName="$Resources:core,All_Documents;" DefaultView="TRUE" DefaultViewForContentType="TRUE" MobileView="True" MobileDefaultView="True" SetupPath="pages\viewpage.aspx" ImageUrl="/_layouts/images/dlicon.png" Url="Forms/AllItems.aspx"><br />  &lt;XslLink Default="true">custom_views.xsl&lt;/XslLink><br />  ...<br /> &lt;/View><br /></pre><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">Here is a sample Visual Studio 2010 solution structure.</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div class="separator" style="clear: both; font-family: inherit; text-align: center;"><span style="font-size: small;"><a href="http://1.bp.blogspot.com/-MuMFr9-FTik/Txmnwa5aOjI/AAAAAAAABK8/Jcw7_OMmvrA/s1600/vs_solution_for_upload.PNG" imageanchor="1"><img border="0" height="320" src="http://1.bp.blogspot.com/-MuMFr9-FTik/Txmnwa5aOjI/AAAAAAAABK8/Jcw7_OMmvrA/s320/vs_solution_for_upload.PNG" width="194" /></a></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">The results (with Telerik Silverlight upload component) are shown below.</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div class="separator" style="clear: both; font-family: inherit; text-align: center;"><span style="font-size: small;"><a href="http://1.bp.blogspot.com/-Nbdk3Fusypg/Txml_XU22CI/AAAAAAAABKk/Hcdeda_T81s/s1600/custom_upload_1.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="132" src="http://1.bp.blogspot.com/-Nbdk3Fusypg/Txml_XU22CI/AAAAAAAABKk/Hcdeda_T81s/s400/custom_upload_1.PNG" width="400" /></a></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div class="separator" style="clear: both; font-family: inherit; text-align: center;"><span style="font-size: small;"><a href="http://1.bp.blogspot.com/-0bG9Mep_vcY/Txml_660Y9I/AAAAAAAABKs/Qie7Qf4puSY/s1600/custom_upload_2.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="207" src="http://1.bp.blogspot.com/-0bG9Mep_vcY/Txml_660Y9I/AAAAAAAABKs/Qie7Qf4puSY/s400/custom_upload_2.PNG" width="400" /></a></span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">This satisfies all my requirements: it's a full programmatic server side solution, it does not require any javascript tricks (high risk that in next version of SP will not work, when for example the css style names will change), it can be applied selectively to specific Document Libraries based on custom list templates (no need to change the default behavior on entire WFE) and it is simple.</span></div><div style="font-family: inherit;"><span style="font-size: small;">Flawless victory :-)</span></div><div style="font-family: inherit;"><span style="font-size: small;"><br /></span></div><div style="font-family: inherit;"><span style="font-size: small;">I hope this short article will spare your time.</span></div>  <br/><b>The VS solution with code samples for this article is available on GoogleCode SVN: <a href="http://byteloom-codesamples.googlecode.com/svn/trunk/LargeFileUploadWithSLToSP/" target="_blank">http://byteloom-codesamples.googlecode.com/svn/trunk/LargeFileUploadWithSLToSP/</a></b>