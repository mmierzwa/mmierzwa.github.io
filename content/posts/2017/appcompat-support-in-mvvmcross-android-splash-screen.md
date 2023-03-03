---
categories:
- mobile
date: "2017-04-20T00:00:00Z"
tags:
- mvvmcross
- android
title: AppCompat support in MVVMCross Android splash screen
---

As it's mentioned in [MVVMCross documentation](https://www.mvvmcross.com/documentation/fundamentals/navigation/navigation.html) Android is quite specific in terms of navigation requirements. The entry point is statically indicated by `MainLauncher = true` attribute parameter on the activity. On rest of the platforms, specifically on iOS, this can be done dynamically by implementing `IMvxAppStart` class for registration in `MvxApplication.Initialize()`.

To achieve this MVVMCross introduce a special lightweight type of activity - `IMvxAndroidSplashScreenActivity`. Unfortunately the default implementation does not support the features from `MvvmCross.Droid.Support.V7.AppCompat` like vector images or material design themes.


The workaround is pretty straightforward. [MvxSplashScreenActivity](https://github.com/MvvmCross/MvvmCross/blob/develop/MvvmCross/Droid/Droid/Views/MvxSplashScreenActivity.cs) can be rewritten to derive from `MvxAppCompatActivity` instead of `MvxActivity`:

```csharp
public abstract class MvxSplashScreenAppCompatActivity
  : MvxAppCompatActivity<MvxNullViewModel>, IMvxAndroidSplashScreenActivity
{
      // ...
}
```

For rest of the code see the [original class](https://github.com/MvvmCross/MvvmCross/blob/develop/MvvmCross/Droid/Droid/Views/MvxSplashScreenActivity.cs).

Happy coding!
