---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-1311222884090176066
blogger_orig_url: http://byteloom.blogspot.com/2012/04/sharepoint-2010-batch-edit.html
date: "2012-04-06T09:46:00Z"
tags:
- sharepoint 2010
title: SharePoint 2010 Batch Edit
---

For the years I have developed .Net projects I've found many useful solutions on [CodePlex](http://www.codeplex.com/) repository. Some of them are only drafts of ideas but there are also many products that are ready to use on production environments. [SharePoint 2010 Batch Edit](http://sp2010batchedit.codeplex.com/) which worked with recently is one of this from the second category.  

As the name suggests, SharePoint 2010 Batch Edit fills the gap in mass updates functionality in SharePoint 2010. In OOB SP2010 you can select one or more items on list view and perform such operations as check-in, check-out, delete an so on, but there is no tool that would allow to update items data for more than one item at once. Of course you can always write your own application page and display it in dialog box but why you should do this if someone already has done it :-)  
<!--more-->
The feature works quite intuitively. After installation from PowerShell (or stsadm) and activation in administration central you will see a new button in items ribbon. When you select more than one item on list the button is enabled and after click you will see a dialog box with a form that allows you update your items.  

<figure class="half center">
  <a href="/images/2012/04/single_file_selected.png" class="image-popup">
	 <img src="/images/2012/04/single_file_selected.png">
   </a>
</figure>

<figure class="half center">
  <a href="/images/2012/04/multiple_files_selected.png" class="image-popup">
	 <img src="/images/2012/04/multiple_files_selected.png">
   </a>
</figure>

I've tested Batch Edit with some basic field types added to lists content type: text, date, user, managed meta-data (MMD) and single value lookup. I haven't experienced any problems with updating values of those types. There are two MMD fields update modes for multi-value fields: overwrite and append, which is quite useful (you can turn on the append mode with check-box on the top of the update form). I had a problem with updating multi-value lookup field - the field control was displayed correctly in form but no changes were applied on update.  

<figure class="half center">
  <a href="/images/2012/04/batch_update_simple.png" class="image-popup">
	 <img src="/images/2012/04/batch_update_simple.png">
   </a>
</figure>

One thing could be confusing at first time. When you are looking for some field that is displayed in edit form for single item it will happen sometimes that it is not available for editing in batch mode. The good example is a standard field - Title in document libraries. The reason of this behavior is that the field must marked as `ShowInNewForm` and `ShowInEdit` form (these conditions are not meet for the Title field in doc. library). When you will dig into the [code](http://sp2010batchedit.codeplex.com/SourceControl/changeset) (`TamTam.SP2010.BatchEdit.Layouts.TamTam.SP2010.BatchEdit.BatchEdit` class) you will find out that also other field types are excluded - computed, file, integer and secondary lookup ([SPField.CanBeDisplayedInEditForm](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spfield.canbedisplayedineditform.aspx) - will return false for them).  

If your list contain more than one content type Batch Edit will display field controls for all your content types. The change of content type itself is however not possible.  

For lists with versioning turned on the update will cause check-out and check-in with new minor version. If some error will occur during the update (like trying to update value for a field marked as unique) you will see a clear message on the bottom of the form.  

I have noticed also a few minor issues using the Batch Edit, mostly in usability area. The first was loosing the focus (checked check-boxes) from previously selected list items after update. This could be annoying for someone that is trying to perform more than one batch update for the same set of elements on the list. For long-time operations (mass updates can take some time) the update form seems to be unresponsive. There should be some progress indicator that will inform the user that update is being performed. Also disabling the update and cancel button in such cases would be a nice thing. The last little bug I've found is that Batch Edit ignores "Launch forms in dialog" option set false (it's a setting that you can change for the list). The update form is also displayed in dialog box.  

PS: If you faced with the problem of updating more than 100 items at once, read my previous post - ["You cannot select more than 100 items at once" in SharePoint 2010]({{ site.baseurl }}{% post_url 2012/2012-03-27-you-cannot-select-more-than-100-items %}).  

Happy SharePointing!
