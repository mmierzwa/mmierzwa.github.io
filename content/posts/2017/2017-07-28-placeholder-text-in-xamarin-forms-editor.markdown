---
categories:
- mobile
date: "2017-07-28T00:00:00Z"
original_url: https://solidbrain.com/2017/07/10/placeholder-text-in-xamarin-forms-editor/
source_page_name: Solidbrain Blog
source_page_url: https://solidbrain.com/blog/
tags:
- xamarin forms
- renderers
- ios
- android
title: Placeholder text in Xamarin.Forms Editor
---

Standard Xamarin.Forms [Xamarin.Forms.Editor control](https://developer.xamarin.com/api/type/Xamarin.Forms.Editor) offers edit capabilities similar to `Entry` but for multiline text. Unfortunately unlike `Entry` it doesn't support displaying placeholder text out of the box. Implementing this functionality with custom renderers can be tricky. Let's see how to do this on Android and iOS.<!--more-->

First step, as for every custom renderer, is to create a custom Forms control:

<script src="https://gist.github.com/mmierzwa/618358f58ec200ee689b2626963d9c32.js"></script>

Internally Android `Editor` renderer uses `EditText` control. It natively supports placeholder text so the implementation is pretty simple:

<script src="https://gist.github.com/mmierzwa/252bfaae4c8db12a358a42473283c002.js"></script>

iOS implementation is more tricky. The internal `Editor` control is `UITextView` which doesn't provide such functionality. This behavior can be imitated in at least two different ways.  
The first is to display the placeholder simply as the same text as the standard control content. This can be hard to style differently than standard text. Also changes detection can lead to bugs (i.e. if user enters the same text as the placeholder).  
The second one is to introduce additional label which will act as the placeholder. Here is the sample code:

<script src="https://gist.github.com/mmierzwa/6989929b33dbd690cce7276cad4cffa7.js"></script>

The placeholder appearance logic is handled by `UITextView` `Changed` and `Ended` events. Unsubscribing from those events is important at control disposal to prevent memory leaks.

I decided to use [Cirrious.FluentLayouts](https://github.com/FluentLayout/Cirrious.FluentLayout) library to simplify the controls positioning. Feel free to choose whatever approach works for you in this matter.

Happy coding!
