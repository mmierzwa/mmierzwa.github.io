---
categories:
- mobile
date: "2018-02-07T00:00:00Z"
tags:
- xamarin forms
- ios
- android
title: Setting tint color in Xamarin.Form image
---

Today's mobile apps are rarely created as text-only. Most of them needs at least in-app icons for toolbars. In many cases you can find graphics for mobile platforms as ready to use resources on the Internet, i.e. [Material Design icons](https://material.io/icons/). Sometimes they are prepared by graphic designers specially for your apps.

No matter which case is your's, sooner or later you will probably need to adjust the images inside your app. One of the most common customisation is setting the tint color of the image. In Xamarin.Forms you can easily do it with [Effects](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/effects/introduction/), which I will show in this post.
<!--more-->

The inspiration for code samples shown bellow was [this tutorial project](https://github.com/shrutinambiar/xamarin-forms-tinted-image). The original implementation was based on [Custom Renderers](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/custom-renderer/) rather than Effects. I personally prefer the later because I can apply many Effects on a common Forms element, mixing the visual changes and added behaviours which I find cumbersome with Custom Renderers approach.

Let's start with the PCL part:

```csharp
using Xamarin.Forms;

namespace Core.Effects
{
    public class TintImageEffect : RoutingEffect
    {
        public const string GroupName = "MyCompany";
        public const string Name = "TintImageEffect";

        public Color TintColor { get; set; }

        public TintImageEffect() : base($"{GroupName}.{Name}") { }
    }
}
```

As you probably already guessed, the `TintColor` is responsible for adjusting the image tint. If you set the alpha channel to fully transparent the tint will obviously be invisible.

The tricky part of iOS Effect implementation is setting the image rendering mode to `AlwaysTemplate`:

```csharp
using System;
using System.Linq;
using UIKit;
using Xamarin.Forms.Platform.iOS;
using FormsTintImageEffect = Core.Effects.TintImageEffect;

namespace iOS.Renderers
{
    public class TintImageEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            try
            {
                var effect = (FormsTintImageEffect) Element.Effects.FirstOrDefault(e => e is FormsTintImageEffect);

                if (effect == null || !(Control is UIImageView image))
                    return;

                image.Image = image.Image.ImageWithRenderingMode(UIImageRenderingMode.AlwaysTemplate);
                image.TintColor = effect.TintColor.ToUIColor();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine($"An error occurred when setting the {typeof(TintImageEffect)} effect: {ex.Message}\n{ex.StackTrace}");
            }
        }

        protected override void OnDetached() { }
    }
}
```

The Android implementation is based on standard `PorterDuffColorFilter`:

```csharp
using System.Linq;
using Android.Graphics;
using Android.Widget;
using Java.Lang;
using Xamarin.Forms.Platform.Android;
using FormsTintImageEffect = Core.Effects.TintImageEffect;

namespace Droid.Renderers
{
    public class TintImageEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            try
            {
                var effect = (FormsTintImageEffect) Element.Effects.FirstOrDefault(e => e is FormsTintImageEffect);

                if (effect == null || !(Control is ImageView image))
                    return;

                var filter = new PorterDuffColorFilter(effect.TintColor.ToAndroid(), PorterDuff.Mode.SrcIn);
                image.SetColorFilter(filter);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine(
                    $"An error occurred when setting the {typeof(TintImageEffect)} effect: {ex.Message}\n{ex.StackTrace}");
            }
        }

        protected override void OnDetached() { }
    }
}
```

In order to enable platform Effects you must register them on both Android:

```csharp
using Droid.Renderers;
using Xamarin.Forms;

// Effects:
[assembly: ResolutionGroupName(Core.Effects.LinkButtonEffect.GroupName)]
[assembly: ExportEffect(typeof(TintImageEffect), Core.Effects.TintImageEffect.Name)]
```

and iOS:

```csharp
using iOS.Renderers;
using Xamarin.Forms;

[assembly:ResolutionGroupName (Core.Effects.BorderEditorEffect.GroupName)]
[assembly:ExportEffect (typeof(TintImageEffect), Core.Effects.TintImageEffect.Name)]
```

If you already have some Effects registered with `assembly:ResolutionGroupName` attribute remember that this needs to be done only once per specific group name. Otherwise you will get an runtime error.

Finally you can consume the new effect in your XAML:

```xml
<!-- xmlns:effects="clr-namespace:Core.Effects;assembly=Core" -->

<Image ...>
    <Image.Effects>
        <effects:TintImageEffect TintColor="Red" />
    </Image.Effects>
</Image>
```

Happy coding!
