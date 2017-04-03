---
layout: post
title: Detecting device resolution in Xamarin apps
date: 2017-03-11 18:00:00 +0200
tags: [xamarin, ios]
categories: [mobile]
---

Considering Xamarin there can be many reasons for need of screen resolution detection in mobile app. You may have more complex logic of loading your resources possibly split between PCL and Android/iOS projects. Other might want to send this information with REST request for reporting. Whatever your reason is, here is a very short text on how to do it in IoC-friendly way.<!-- more -->

First we will need some abstraction for the platform-agnostic part of the solution:

```csharp
public interface ISystemInfoService
{
    Platform Platform { get; }
    Resolution Resolution { get; }
}
```

Both fields are enums. `Platform` is obviously a platform name. `Resolution` is combined set of all resolutions from both Android and iOS:

```csharp
public enum Platform
{
    Android,
    iOS
}

public enum Resolution
{
    Ldpi,
    Mdpi,
    Hdpi,
    Xhdpi,
    Xxhdpi,
    Xxxhdpi,
    X1,
    X2,
    X3
}
```

Of course you may want to prefer bare strings over the enums.

Last two parts are the platform-specific providers that you will bind in your DI configuration.  
In Android version I'm using the MvvmCross helper class - `IMvxAndroidCurrentTopActivity` - to obtain reference to app resources. This can be achieved in many ways:

```csharp
public class DroidSystemInfoService : ISystemInfoService
{
    private readonly IMvxAndroidCurrentTopActivity _topActivity;

    public DroidSystemInfoService(IMvxAndroidCurrentTopActivity topActivity)
    {
        _topActivity = topActivity;
    }

    public Platform Platform => Platform.Android;

    public Resolution Resolution
    {
        get
        {
            var density = _topActivity.Activity.Resources.DisplayMetrics.DensityDpi;
            switch (density)
            {
                case DisplayMetricsDensity.Low: return Resolution.Ldpi;
                case DisplayMetricsDensity.Medium: return Resolution.Mdpi;
                case DisplayMetricsDensity.High: return Resolution.Hdpi;
                case DisplayMetricsDensity.Xhigh: return Resolution.Xhdpi;
                case DisplayMetricsDensity.Xxhigh: return Resolution.Xxhdpi;
                case DisplayMetricsDensity.Xxxhigh: return Resolution.Xxxhdpi;
                default: return Resolution.Xhdpi;
            }
        }
    }
}
```

And here is the iOS version:

```csharp
public class IosSystemInfoService : ISystemInfoService
{
    public Platform Platform => Platform.iOS;

    public Resolution Resolution
    {
        get
        {
            switch ((int) UIScreen.MainScreen.Scale)
            {
                case 1: return Resolution.X1;
                case 2: return Resolution.X2;
                case 3: return Resolution.X3;
                default: return Resolution.X3;
            }
        }
    }
}
```

Happy coding!  
Marek
