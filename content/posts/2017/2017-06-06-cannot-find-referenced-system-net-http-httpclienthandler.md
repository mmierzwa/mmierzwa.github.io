---
categories:
- mobile
date: "2017-06-06T00:00:00Z"
tags:
- xamarin
- ios
title: ModernHttpClient and 'Type HttpClientHandler defined in unreferenced assembly'
  error
---

[ModernHttpClient](https://github.com/paulcbetts/ModernHttpClient) is a great wrapper around native HTTP clients offered by Xamarin. It wraps `NSURLSession` on iOS and `OkHttp` on Android. As it deals pretty well with SSL/TLS stack (especially in uncommon scenarios) it's often used instead of built-in types. I switched to `ModernHttpClient` because of  weird errors on connections to the preproduction environment in Android.<!--more-->

One disadvantage of using this library is that it's not actively maintained (last update in repo was from Jan 2016). There are [few annoying bugs](https://github.com/paulcbetts/ModernHttpClient/issues) that have not been (and probably won't be) fixed. Because of [one of them](https://github.com/paulcbetts/ModernHttpClient/issues/195) I was forced to customize the Android client a little bit. The result was the unusual reference to the `NativeMessageHandler` in iOS project. Locally everything was working fine but on the build machine the following error appeared (in the native iOS project):

```
error CS0012: The type `System.Net.Http.HttpClientHandler' is defined in an assembly that is not referenced. Consider adding a reference to assembly `System.Net.Http, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'
```

This error seemed to be similar to the one described on [SO](https://stackoverflow.com/questions/37958107/cannot-load-system-net-http-primitives-in-xamarin-ios). The suggested solution was to re-create the entire solution from bare template. Obviously I'm not a big fan of such methods.

After trying to add few NuGets which seemed natural (`System.Net.Http`, `System.Net.Primitives`) I figured out that this was not the case. Libs from these packages were not those referenced by `ModernHttpHandler`. Good IDE, such as [JetBrains Rider](https://www.jetbrains.com/rider/), with decompiler integrated with the smart navigation can be really helpful in such investigations.

The solution was adding reference to `System.Net.Http` assembly from standard Mono libs directly in iOS csproj file:

```xml
<Reference Include="System.Net.Http" />
```

Happy coding!
