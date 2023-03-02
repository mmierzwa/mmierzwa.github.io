---
categories:
- mobile
date: "2017-06-07T00:00:00Z"
tags:
- xamarin
- ios
- forms
title: Disabled button style on iOS in Xamarin.Forms
---

All the credits for solution described in this post goes [my friend qbus](https://www.facebook.com/qbus00). He saved few hours of my life and my sanity ;-)

In Xamarin.Forms application I have a button with non-default styling (defined in XAML). Target platforms are iOS and Android. Enabled and disabled states should obviously be distinguished with colors.

The most straightforward solution that makes use of [Data Triggers](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/triggers/) works well for Android. For some reason it doesn't want to work properly on iOS. If the button is initially disabled it displays correctly in this state but after changing `IsEnabled` to `true` and then back to `false` it does not anymore. Interestingly the background taken from style setter is good, but not the text color property. It turns out to be set to some awkward value: `rgb(123, 123, 123)` with alpha `89`. I guess this is the default value for disabled state because it's not defined anywhere in the app.

The problem is well described [on the Xamarin Forum thread](https://forums.xamarin.com/discussion/40830/disabled-button-color). Unfortunately none of the proposed workarounds worked for me.

The initial workaround implemented by the previous developer was implementing a custom renderer. In `OnElementChanged(ElementChangedEventArgs<Button>)` the button's title color was set for `UIControlState.Disabled` state. For some weird reason this was working only for the initial disable state (as described above). The second solution, proposed by my friend, is using the `NSAttributedString` set as entire button title:

```csharp
public class ButtonDisabledTextColorRenderer : ButtonRenderer
{
    private static UIColor DisabledColor => Color.FromHex("#FFFFFF").ToUIColor();

    protected override void OnElementChanged(ElementChangedEventArgs<Button> e)
    {
        base.OnElementChanged(e);

        if (Control == null)
            return;

        var title = new NSAttributedString(Control.CurrentTitle, new UIStringAttributes {ForegroundColor = DisabledColor});
        Control.SetAttributedTitle(title, UIControlState.Disabled);
    }
}
```

I should call this rather workaround than the real clean solution. Xamarin.Forms obviously has some bug in this case. Nevertheless it works.

Happy coding!
