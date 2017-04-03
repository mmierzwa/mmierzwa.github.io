---
layout: post
title: How to add padding to UILabel in Xamain.iOS
date: 2017-02-20 18:00:00 +0200
tags: [xamarin, ios]
categories: [mobile]
---

Working with mobile can be quite challenging for a developer with a web dev background. At least that is my experience so far. Comparing to typical HTML web elements with CSS styling some features might be missing.

An example for this kind of issues I faced recently is lack explicit padding for `UILabel` (and not only). You can either let the label to fit tightly around the text content or set the label's size (either statically or with auto-sizing).

There are few solutions for this around the Internet (like [this](http://stackoverflow.com/a/17557490/619799) or [this](https://forums.xamarin.com/discussion/comment/190767/#Comment_190767)). Most of them are implemented in Swift/Obj-C, some are somehow incomplete even if provided with Xamarin code. Here is a short compilation of my findings.<!-- more -->

The steps are easy:

1. Sub-class `UILabel` class
2. Override `DrawText(CGRect)` method
3. Override `CGSize IntrinsicContentSize` property - if you miss this one your content will probably not fit (result will depend on the `UIView.ContentMode`)

Fortunately both method and property are implemented as virtual.

Here is the ready example:

```csharp
public class UiExtraPaddingLabel : UILabel{    private readonly UIEdgeInsets _edgeInsets;    public UiExtraPaddingLabel(nfloat padding)        : this(padding, padding, padding, padding)    {    }    public UiExtraPaddingLabel(nfloat top, nfloat left, nfloat bottom, nfloat right)    {        _edgeInsets = new UIEdgeInsets(top, left, bottom, right);    }    public override void DrawText(CGRect rect)    {        base.DrawText(_edgeInsets.InsetRect(rect));    }    public override CGSize IntrinsicContentSize    {        get        {            var originalSize = base.IntrinsicContentSize;            originalSize.Width += _edgeInsets.Left + _edgeInsets.Right;            originalSize.Height += _edgeInsets.Top + _edgeInsets.Bottom;            return originalSize;        }    }}
```

Cheers!
Marek
