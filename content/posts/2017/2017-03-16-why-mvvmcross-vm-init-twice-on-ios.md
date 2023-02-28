---
categories:
- mobile
date: "2017-03-16T18:00:00Z"
tags:
- xamarin
- mvvmcross
- ios
title: Why does MVVMCross view model initialize twice on iOS?
---

Working on a bug fix in MVVMCross-based mobile application I noticed a strange behavior. The navigation to other view model I put in `async Init<TInit>(TInit parameters)` which as executed on the first view model in my app was running twice. After a short debugging session it turned out that `MvxViewModel<TInit>` `Init()` is called from the view controllers `ViewDidLoad()` method. Obviously there was something I was missing in terms of `ViewDidLoad()` semantics.<!--more-->

Quick search on SO reviled the mystery of iOS SDK (most relevant questions are [here](http://stackoverflow.com/questions/26875936/why-is-viewdidload-being-called-twice) and [here](http://stackoverflow.com/questions/7079602/viewdidload-is-called-twice)). The `ViewDidLoad()` is not considered to be "safe" in any other aspect than just updating the UI. It can be run any number of times on loading the view. So if you, like me, have some logic that should be run only once you need to put it somewhere else.

Unfortunately my `Init()` from VM could not be moved somewhere else easily. I could try to move the logic to some form of queued tasks with "run-only-once" semantics. The other choice, which I actually took, was preventing the double-run in the VM itself. This can be done by introducing the guard flag and checking it in `Init()`:

```csharp
public class MyViewModel : MvxViewModel<SomeInitType> ...
  bool IsInitialized;
  ...
  protected Init<SomeInitType>(SomeInitType parameters)
  {
    if (IsInitialized)
    {
      return;
    }

    // do the initialization here
  }
```

Simple as that. Actually I moved this to my base view model class so other VMs could use it (but still decide whenever to allow for multiple init).

Happy coding!  
Marek
