---
categories:
- mobile
date: "2018-06-18T00:00:00Z"
tags:
- xamarin forms
- xamarin
title: Implementing search as type in Xamarin.Forms search bar
---

Search as type functionality is quite often seen on the web as well as in mobile apps. Let's see how to make it work in Xamarin.Forms.


The simplest way of implementing such behavior is to run a search on each phase change. In other words - every time user types or deletes a letter in a search box the search function (and results list update) is performed. As you may already see, this is not the most efficient way of doing this.

Usually, when the user changes the search phrase the search is delayed for a short timespan (less than 500ms). If in the meantime next search phrase change occurs the search for the previous phrase is canceled and delay timer starts again. When the delay is not interrupted by further search phrase changes the actual search is being run.

I personally find using Xamarin.Forms behaviors more elegant than hooking-up the event directly to controls (on page level). It's also more flexible than writing custom controls. Actually creating custom controls and custom renderers should be your last choice (if you can achieve the same with [Behavior](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/behaviors/) or [Effect](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/effects/)).

Here is the example of behavior that can be applied to Forms `SearchBar`.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Input;
using Behaviors;
using Xamarin.Forms;

namespace MyApp
{
    public class SearchAsYouTypeBehavior : BehaviorBase<SearchBar>
    {
        public const int DefaultMinimumSearchIntervalMiliseconds = 300;

        private CancellationTokenSource _cancellationTokenSource;

        public static readonly BindableProperty SearchCommandProperty =
            BindableProperty.Create(nameof(SearchCommand), typeof(ICommand), typeof(SearchAsYouTypeBehavior),
                propertyChanged: SearchCommandChanged);

        public static readonly BindableProperty MinimumSearchIntervalMilisecondsProperty =
            BindableProperty.Create(nameof(MinimumSearchIntervalMiliseconds), typeof(int),
                typeof(SearchAsYouTypeBehavior), DefaultMinimumSearchIntervalMiliseconds);

        public ICommand SearchCommand
        {
            get => (ICommand) GetValue(SearchCommandProperty);
            set => SetValue(SearchCommandProperty, value);
        }

        public int MinimumSearchIntervalMiliseconds
        {
            get => (int) GetValue(MinimumSearchIntervalMilisecondsProperty);
            set => SetValue(MinimumSearchIntervalMilisecondsProperty, value);
        }

        protected override void OnDetachingFrom(SearchBar bindable)
        {
            base.OnDetachingFrom(bindable);
            AssociatedObject.TextChanged -= Search;
        }

        private static void SearchCommandChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var behavior = (SearchAsYouTypeBehavior) bindable;
            behavior.SearchCommandChanged(newValue);
        }

        private void SearchCommandChanged(object newCommand)
        {
            if (newCommand is ICommand)
                AssociatedObject.TextChanged += Search;
            else
                AssociatedObject.TextChanged -= Search;
        }

        private async void Search(object sender, TextChangedEventArgs textChangedEventArgs)
        {
            _cancellationTokenSource?.Cancel();

            _cancellationTokenSource = new CancellationTokenSource();
            var cancellationToken = _cancellationTokenSource.Token;

            try
            {
                await Task.Delay(MinimumSearchIntervalMiliseconds, cancellationToken);

                if (cancellationToken.IsCancellationRequested)
                    return;

                ExecuteSearch();
            }
            catch (OperationCanceledException)
            {
                // swallow
            }
        }

        private void ExecuteSearch()
        {
            Device.BeginInvokeOnMainThread(() =>
            {
                SearchCommand?.Execute(null);

                if (!AssociatedObject.IsFocused)
                    AssociatedObject.Focus();
            });
        }
    }
}
```

Since it defines `ICommand` as a bindable property it can be binded in standard MVVM way in XAML:

```xml
<SearchBar SearchCommand="{Binding SearchCommand}"
           Text="{Binding SearchQuery}">
    <SearchBar.Behaviors>
        <behaviors:SearchAsYouTypeBehavior SearchCommand="{Binding SearchCommand}" />
    </SearchBar.Behaviors>
</SearchBar>
```

You can also specify `MinimumSearchIntervalMiliseconds` to decide how long should be the delay between searches.

[In next post](/posts/search-as-you-type-in-xamarin-forms-part-2), I'll show you how to implement this behavior even simpler and more efficiently with [Reactive Extensions (Rx)](https://github.com/dotnet/reactive).

Happy coding!
