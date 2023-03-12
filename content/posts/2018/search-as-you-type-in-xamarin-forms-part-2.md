---
categories:
- mobile
date: "2018-06-23T00:00:00Z"
tags:
- xamarin forms
- xamarin
title: Search as you type in Xamarin.Forms - the Reactive Extensions way
---

In [my previous post](/blog/search-as-you-type-in-xamarin-forms) I showed how to introduce search as type behavior into Xamarin.Forms app with standard [Forms Behaviors](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/behaviors/).

Let's see how to do it in other, more declarative and configurable manner, using [Reactive Extensions (Rx)](https://github.com/dotnet/reactive) instead of `Task` and `CancellationTokenSource`.

To keep the code as short as possible I'll further assume that you have read the [previous article](/blog/search-as-you-type-in-xamarin-forms) and have it opened as a reference. The implementation uses the same Forms `Behavior` as the previous one did, but the delay between search phrase changes and running the search will be handled differently.

With Reactive Extensions (Rx) you don't handle the events directly. Instead, you are using an [observer design pattern](https://en.wikipedia.org/wiki/Observer_pattern) with Rx [Observable](http://reactivex.io/documentation/observable.html) object. This object is responsible for watching the source changes and reacting in a defined way.

In our case, the source is the `SearchBar` and changes are represented by `TextChanged` events. To create `Observable` from events Rx uses `Observable.FromEventPattern()` method. It handles both adding and removing the handlers (on disposal):

```csharp
private readonly IDisposable _subscription;

public SearchAsYouTypeBehavior()
{
    _subscription = Observable.FromEventPattern<TextChangedEventArgs>(
            handler => AssociatedObject.TextChanged += handler,
            handler => AssociatedObject.TextChanged -= handler);
    // ...
}
```

In the original concept, the delay was implemented with standard `Task` cancellation pattern. Rx is actually designed exactly for this kind of behaviors. To introduce a delay simply use `Observable.Throttle()` method:

```csharp
_subscription.Throttle(TimeSpan.FromMilliseconds(MinimumSearchIntervalMiliseconds));
```

Next we will have to ensure that all further operations are performed on the UI thread:

```csharp
_subscription.ObserveOn(SynchronizationContext.Current);
```

Rx provides a convenient way of dealing the event streams with Linq. We can use the standard operators, such as `Select` or `Where` to transform the original object representing changes in virtually any way you wish. We will use this feature to select the search phrase that will be used in further processing:

```csharp
_subscription.Select(eventPattern => AssociatedObject.Text)
             .DistinctUntilChanged();
```

The `DistinctUntilChanged()` method, in this case, ensures that observable subscription will not be run until the selected text has really been changed. A user can, for example, type and immediately delete a letter. Using `DistinctUntilChanged()` will prevent recognizing this as a change.

Finally, here is the actual search run configuration:

```csharp
_subscription.Subscribe(query => Device.BeginInvokeOnMainThread(() =>
    {
        SearchCommand?.Execute(query);
    })
);
```

Of course you can perform the entire configuration in single fluent expression. I broke down the code just for the better description of each step.

Reactive Extensions is an amazing tool which you can use solve many common problems in mobile apps development, like background fetching or recurrent refresh. Check out the [101 Rx Samples](http://rxwiki.wikidot.com/101samples) for more examples.

Happy coding!
