---
layout: post
title: '"You cannot select more than 100 items at once" in SharePoint 2010'
date: 2012-03-27 15:49:00 +0200
tags: [sharepoint 2010]
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-273667762726925100
blogger_orig_url: http://byteloom.blogspot.com/2012/03/you-cannot-select-more-than-100-items.html
---

Looking for some solution for batch items update I've run into the following problem: when I clicked a checkbox in top right corner of the list (actually a document library) "All Items" view, the browser displayed an error: `You cannot select more than 100 items at once.`

<figure class="half center">
  <a href="/images/2012/03/items_selection_limit_error.png" class="image-popup">
	 <img src="/images/2012/03/items_selection_limit_error.png" alt="Items selection limit - error message">
   </a>
</figure>

The problem was discussed on the TechNet forum ([Selecting more than 100 files in Document Library](http://social.technet.microsoft.com/Forums/en-US/sharepoint2010general/thread/9c99cc7e-68d0-43f6-bd7b-ca5b54f2d5bf)) and the limit of maximum 100 items per one batch update (like check-in, delete etc.) is described [on MSDN](http://technet.microsoft.com/en-us/library/cc262787.aspx#ListLibrary):  

> Limit: Bulk operations  
> Maximum value: 100 items per bulk operation  
> Limit type: Boundary  
> Notes: The user interface allows a maximum of 100 items to be selected for bulk operations.

What is interesting and what was not mentioned in TechNet topic, the limit is controlled on client side by a script, what I will show.
<!-- more -->What happens when you click on select-all-items check box? This is a pseudo-callstack of JS calls:  

```html
<input type="checkbox"   
       title="Select or deselect all items"   
       onclick="ToggleAllItems(event,this,21)"   
       onfocus="EnsureSelectionHandlerOnFocus(event,this,21)" />  
```

```js
onClick (the view HTML) ->  
ToggleAllItems(evt, cbx, ctxNum) (init.js) ->  
CoreInvoke(fn) (init.js) ->  
_ToggleAllItems(evt, cbx, ctxNum) (core.js) ->  
ToggleAllItems2(cbx, ctxNum, f) (core.js) ->  
```

In the last function - `ToggleAllItems2` - you will find the following pieces of code:  

```js
var totalItems=CountTotalItems(ctxCur);  
// ...  
if (totalItems > g_MaximumSelectedItemsAllowed)  
{  
    cbx.checked=false;  
    alert(L_BulkSelection_TooManyItems);  
    return;  
}  
```

Variables `g_MaximumSelectedItemsAllowed` and `L_BulkSelection_TooManyItems` are defined in the same file (`core.js`):  

```js
var g_MaximumSelectedItemsAllowed=100;  
var L_BulkSelection_TooManyItems="You cannot select more than 100 items at once.";  
```

You will find a similar code checking if the limits are also preserved when user checks items one by one.  
The second interesting thing is that the number of items is checked only on client side, so if you change `g_MaximumSelectedItemsAllowed` value to, let say 200, you will be able to make a batch update for more items than the official limit allows.  
I've managed to change this by IE developers toolbar for 200 items:  

<figure class="half center">
  <a href="/images/2012/03/maximum_selected_items_limit_overwrite_3.png" class="image-popup">
	 <img src="/images/2012/03/maximum_selected_items_limit_overwrite_3.png" alt="Maximum selected items limit overwrite">
   </a>
</figure>

Of course if you want to change this permanently (probably with changing the message string `L_BulkSelection_TooManyItems` also to not confuse users) you still will have to develop some more persistent solution - i.e. deploy your custom script into `Layouts` folder, link it on selected pages and so on. The code would look like this (I assume you have jQuery included):  

```js
$(document).ready(function() {  
    g_MaximumSelectedItemsAllowed = 200;  
    L_BulkSelection_TooManyItems = "You cannot select more than ' + g_MaximumSelectedItemsAllowed + ' items at once.";  
});  
```

There is one other issue I should mention. Limit of 100 items has been set for some reason - probably the performance. I've managed to update value of a column for 200 items with [BatchEdit from codeplex](http://sp2010batchedit.codeplex.com) but checking-in the same number of items failed (I've got some strange JS error). Still I was able to check-in 110 items which was above the standard limit.  

Happy SharePointing!
