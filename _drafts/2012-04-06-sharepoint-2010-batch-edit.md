---
layout: post
title: SharePoint 2010 Batch Edit
date: '2012-04-06T09:46:00.000+02:00'
author: Marek Mierzwa
tags:
- sharepoint
- batch update
- sharepoint 2010
- edit form
modified_time: '2012-04-06T09:46:48.639+02:00'
thumbnail: http://1.bp.blogspot.com/-DeAITAMTFyg/T36eMcexMmI/AAAAAAAABQ4/zv5zuPOpwvU/s72-c/single_file_selected.PNG
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-1311222884090176066
blogger_orig_url: http://byteloom.blogspot.com/2012/04/sharepoint-2010-batch-edit.html
---

For the years I have developed .Net projects I've found many useful solutions on <a href="http://www.codeplex.com/" target="_blank">CodePlex</a> repository. Some of them are only drafts of ideas but there are also many products that are ready to use on production environments. <a href="http://sp2010batchedit.codeplex.com/" target="_blank">SharePoint 2010 Batch Edit</a> which worked with recently is one of this from the second category.<br /><br />As the name suggests, SharePoint 2010 Batch Edit fills the gap in mass updates functionality in SharePoint 2010. In OOB SP2010 you can select one or more items on list view and perform such operations as check-in, check-out, delete an so on, but there is no tool that would allow to update items data for more than one item at once. Of course you can always write your own application page and display it in dialog box but why you should do this if someone already has done it :-)<br /><a name='more'></a><br />The feature works quite intuitively. After installation from PowerShell (or stsadm) and activation in administration central you will see a new button in items ribbon. When you select more than one item on list the button is enabled and after click you will see a dialog box with a form that allows you update your items.<br /><br /><div class="separator" style="clear: both; text-align: center;"><a href="http://1.bp.blogspot.com/-DeAITAMTFyg/T36eMcexMmI/AAAAAAAABQ4/zv5zuPOpwvU/s1600/single_file_selected.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="145" src="http://1.bp.blogspot.com/-DeAITAMTFyg/T36eMcexMmI/AAAAAAAABQ4/zv5zuPOpwvU/s320/single_file_selected.PNG" width="320" /></a></div><br /><div class="separator" style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-DllFQXM8_UY/T36eL8p9uCI/AAAAAAAABQw/gXB6TZn5xLc/s1600/multiple_files_selected.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="145" src="http://3.bp.blogspot.com/-DllFQXM8_UY/T36eL8p9uCI/AAAAAAAABQw/gXB6TZn5xLc/s320/multiple_files_selected.PNG" width="320" /></a></div><br />I've tested Batch Edit with some basic field types added to lists content type: text, date, user, managed meta-data (MMD) and single value lookup. I haven't experienced any problems with updating values of those types. There are two MMD fields update modes for multi-value fields: overwrite and append, which is quite useful (you can turn on the append mode with check-box on the top of the update form). I had a problem with updating multi-value lookup field - the field control was displayed correctly in form but no changes were applied on update.<br /><br /><div class="separator" style="clear: both; text-align: center;"><a href="http://4.bp.blogspot.com/-J6sBWYWuIkw/T36eKkEpzVI/AAAAAAAABQg/QcW9EehECds/s1600/batch_update_simple.PNG" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="150" src="http://4.bp.blogspot.com/-J6sBWYWuIkw/T36eKkEpzVI/AAAAAAAABQg/QcW9EehECds/s320/batch_update_simple.PNG" width="320" /></a></div><br />One thing could be confusing at first time. When you are looking for some field that is displayed in edit form for single item it will happen sometimes that it is not available for editing in batch mode. The good example is a standard field - Title in document libraries. The reason of this behavior is that the field must marked as ShowInNewForm and ShowInEdit form (these conditions are not meet for the Title field in doc. library). When you will dig into the <a href="http://sp2010batchedit.codeplex.com/SourceControl/changeset" target="_blank">code</a> (TamTam.SP2010.BatchEdit.Layouts.TamTam.SP2010.BatchEdit.BatchEdit class) you will find out that also other field types are excluded - computed, file, integer and secondary lookup (<a target="_blank" href="http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spfield.canbedisplayedineditform.aspx">SPField.CanBeDisplayedInEditForm</a> - will return false for them).<br /><br />If your list contain more than one content type Batch Edit will display field controls for all your content types. The change of content type itself is however not possible.<br /><br />For lists with versioning turned on the update will cause check-out and check-in with new minor version. If some error will occur during the update (like trying to update value for a field marked as unique) you will see a clear message on the bottom of the form.<br /><br />I have noticed also a few minor issues using the Batch Edit, mostly in usability area. The first was loosing the focus (checked check-boxes) from previously selected list items after update. This could be annoying for someone that is trying to perform more than one batch update for the same set of elements on the list. For long-time operations (mass updates can take some time) the update form seems to be unresponsive. There should be some progress indicator that will inform the user that update is being performed. Also disabling the update and cancel button in such cases would be a nice thing. The last little bug I've found is that Batch Edit ignores "Launch forms in dialog" option set false (it's a setting that you can change for the list). The update form is also displayed in dialog box.<br /><br />PS: If you faced with the problem of updating more than 100 items at once, read my previous post - <a target="_blank" href="{{ site.baseurl }}{% post_url 2012-03-27-you-cannot-select-more-than-100-items %}">"You cannot select more than 100 items at once" in SharePoint 2010</a>.<br /><br />Happy sharepointing!