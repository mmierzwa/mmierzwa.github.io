---
categories:
- mobile
- developer story
date: "2017-04-04T22:56:00Z"
tags:
- xamarin
- android
title: 'AAPT: Unknown option ''--no-version-vectors'''
---

If you ever encountered the error `AAPT: Unknown option '--no-version-vectors'` during the Xamarin build you probably found [this page](https://forums.xamarin.com/discussion/63482/aapt-error-unknown-option-no-version-vectors) looking for a solution. Like I did. Then you probably first tried setting `AndroidSdkBuildToolsVersion` to the latest you have. Just like me. Or uninstall all the older versions. If this didn't work (like in my case) stay with me.

The suggestion of changing the [build tools](https://developer.xamarin.com/guides/android/under_the_hood/build_process/#AndroidSdkBuildToolsVersion) version in Droid csproj didn't even look promising for me. I had only one version installed - `25.0.3` - and this was the latest one (at the time I'm writing this post). Nevertheless I checked this out, just in case, as this was the accepted solution in this thread. The error did not disappear.

I find a reasonable level of laziness practical. In fact this is usually a motivation for me to automate things. And it often pays off. Unfortunately not this time. As a typical lazy developer I skipped the rest of the discussion in the [forum thread](https://forums.xamarin.com/discussion/63482/aapt-error-unknown-option-no-version-vectors) and started digging on my own. After a wasted hour I got back to the thread and found out that the build version tools solution didn't work for others as well. Importantly [one of later posts](https://forums.xamarin.com/discussion/comment/201553/#Comment_201553) was pointing the solution.

True reason was the duplicated MSBuild target imports in csproj file:

```xml
<Import Project="..\packages\Xamarin.Android.Support.Vector.Drawable.25.1.0\build\MonoAndroid70\Xamarin.Android.Support.Vector.Drawable.targets" Condition="Exists('..\packages\Xamarin.Android.Support.Vector.Drawable.25.1.0\build\MonoAndroid70\Xamarin.Android.Support.Vector.Drawable.targets')" />
<Import Project="..\packages\Xamarin.Android.Support.Animated.Vector.Drawable.25.1.0\build\MonoAndroid70\Xamarin.Android.Support.Animated.Vector.Drawable.targets" Condition="Exists('..\packages\Xamarin.Android.Support.Animated.Vector.Drawable.25.1.0\build\MonoAndroid70\Xamarin.Android.Support.Animated.Vector.Drawable.targets')" />
...
<Import Project="..\packages\Xamarin.Android.Support.Vector.Drawable.25.1.1\build\MonoAndroid70\Xamarin.Android.Support.Vector.Drawable.targets" />
<Import Project="..\packages\Xamarin.Android.Support.Animated.Vector.Drawable.25.1.1\build\MonoAndroid70\Xamarin.Android.Support.Animated.Vector.Drawable.targets" />
```

This was probably caused by NuGet package manager (rather the clone used in Xamarin Studio or JetBrain's Rider I'm using).

Lesson learned - RTFM ;-) And don't be **too** lazy, as I am.

Happy coding!
