---
categories:
- mobile
date: "2017-07-02T00:00:00Z"
tags:
- xamarin forms
- ios
title: Badge icon notification in Xamarin.Forms ToolbarItems on iOS
---

Most of iPhone and iPad users can easily recognize icon badges - the pattern for application notifications typically presented in app icon or navigation bar. People that got used to this pattern might want to have the same user experience in their Xamarin Forms application. This post describes how to customize the navigation toolbar in iOS to dynamically display such elements.<!--more-->

When I was asked to prepare the static (but still clickable and interactive) mockup views that include badge icon in nav bar I didn't realize that the task is really non-trivial. It's the one from category of tasks that are much easier to implement in native Xamarin. In this case the Xamarin Forms add few layers of indirection that makes the process painful.

First thing I learned is that I cannot simply implement the custom renderer for `ToolbarItem` element because there is no publicly available implementation of such. The one on the PCL Forms side is only a view holder or representation of navigation item. On the native side the whole process of building the navigation toolbar is controlled by A `PageRenderer`. If any customization is required the developer needs to implement the [custom page renderer](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/custom-renderer/contentpage/) (and probably the custom base `ContentPage` as well). Fortunately I found this [Xamarin Forum thread](https://forums.xamarin.com/discussion/68303/how-to-show-badge-count-in-toolbar-item) and a [blog post of Jason Farrel](https://jfarrell.net/2017/05/28/toolbar-navigation-in-xamarin-forms/) that helped me to understand the design of this part of Xamarin Forms. They were my starting point.

Second surprise was that I was not able to find any built-in support for in-app badge icons in iOS UIKit. I found few custom 3rd party components written in Objective C or Swift. The perspective of writing the bindings convinced me to implement the badge in Xamarin. I was intensively using [the Gist sample](https://gist.github.com/yonat/75a0f432d791165b1fd6) I found during my research.

I think that's enough for the introduction. Let's see some code!

First thing is the new base `ContentPage`. To be strict this is not absolutely required. In next step you can implement the page renderer that will be applied to all your content pages. Nevertheless I still recommend this. When your app continues to grow you will probably need it anyway to share some properties between your views.

```csharp
using Xamarin.Forms;

namespace Core.Pages.Base
{
    public class ApplicationContentPageBase : ContentPage { }
}
```

As said the second step is to build the custom platform-specific page renderer. Here you must decide how your app will notify the badge rendering logic about changes in badge values. There are several ways to achieve this. One option is to use the bindings and callbacks (events). In this case remember to unsubscribe from all the events at proper time to avoid memory leaks. Keep in mind that the notification mechanism will probably strongly depend on the source of badge values.

I decided to use the Xamarin Forms `MessagingCenter` relay for this communication (together with bindings described later). Renderer is responsible for subscribing, receiving notifications and proper laying out the indicator on navigation button:

```csharp
using System.Diagnostics;
using Cirrious.FluentLayouts.Touch;
using Core.Pages.Base;
using Core.Views;
using iOS.Controls;
using iOS.Renderers;
using UIKit;
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;

[assembly: ExportRenderer(typeof(ApplicationContentPageBase), typeof(ApplicationContentPageBaseRenderer))]
namespace iOS.Renderers
{
    public class ApplicationContentPageBaseRenderer : PageRenderer
    {
        private bool _initialized;

        private ApplicationContentPageBase Page => (ApplicationContentPageBase) Element;

        public override void ViewDidLoad()
        {
            if (_initialized)
                return;

            MessagingCenter.Subscribe<BadgeToolbarItem, int>(this, BadgeToolbarItem.BadgeValueChangedMessage,
                UpdateBadgeValue);
            _initialized = true;

            base.ViewDidLoad();
        }

        public override void ViewWillDisappear(bool animated)
        {
            base.ViewWillDisappear(animated);

            MessagingCenter.Unsubscribe<BadgeToolbarItem, int>(this, BadgeToolbarItem.BadgeValueChangedMessage);
        }

        private void UpdateBadgeValue(BadgeToolbarItem sender, int newValue)
        {
            var navigationItem = NavigationController.TopViewController.NavigationItem;

            var itemIndex = Page.ToolbarItems.Count - Page.ToolbarItems.IndexOf(sender) - 1;
            var item = navigationItem.RightBarButtonItems[itemIndex];

            if (newValue < 1)
            {
                item.CustomView?.Dispose();
                item.CustomView = null;
                return;
            }

            var badge = new UIBadgeLabel(newValue);

            var customItemView = new UIImageView(item.Image);
            customItemView.AddSubview(badge);

            customItemView.SubviewsDoNotTranslateAutoresizingMaskIntoConstraints();
            customItemView.AddConstraints(
                badge.AtTopOf(customItemView),
                badge.AtRightOf(customItemView));

            customItemView.UserInteractionEnabled = true;
            customItemView.AddGestureRecognizer(new UITapGestureRecognizer(() =>
            {
                var parameter = sender.CommandParameter;
                sender.Command.Execute(parameter);
            }));

            item.CustomView = customItemView;
        }
    }
}
```

I added `UITapGestureRecognizer` because after setting `CustomView` property on nav bar button the tap gesture stoped working. To simplify the positioning I used `Cirrious.FluentLayouts` package. I describe the `UIBadgeLabel` later in this text.

The Forms part of the messaging communication is still required. I added this logic to the binding property in custom `ToolbarItem`. You might consider to simplify it by putting this in background refresh logic or somewhere else.

```csharp
using Xamarin.Forms;

namespace Core.Views
{
    public class BadgeToolbarItem : ToolbarItem
    {
        public const string BadgeValueChangedMessage = "MenuItemBadgeValueChanged";

        public static BindableProperty BadgeValueProperty =
            BindableProperty.Create(nameof(BadgeValue), typeof(int), typeof(BadgeToolbarItem), 0, propertyChanged: BadgeValuePropertyChanged);

        public int BadgeValue
        {
            get => (int) GetValue(BadgeValueProperty);
            set => SetValue(BadgeValueProperty, value);
        }

        private static void BadgeValuePropertyChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var item = (BadgeToolbarItem) bindable;

            if (oldValue != newValue)
                MessagingCenter.Send(item, BadgeValueChangedMessage, (int) newValue);
        }
    }
}
```

In order to use the bindable property the view model needs to have a broadcasting property. Here is an example:

```csharp
namespace Core.PageModels.Customers
{
    public class MyCustomersPageModel : INotifyPropertyChanged
    {
        private int _numberOfWarnings;
        public int NumberOfWarnings
        {
            get => _numberOfWarnings;
            set
            {
                _numberOfWarnings = value;
                RaisePropertyChanged();
            }
        }

        // update the number in your VM initialization/proper lifecycle moment/other asynchronous events
    }
}
```

In XAML all those elements can be used like below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<pages:ApplicationContentPageBase xmlns="http://xamarin.com/schemas/2014/forms"
                                  xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                                  xmlns:pages="clr-namespace:Core.Pages.Base"
                                  xmlns:views="clr-namespace:Core.Views"
                                  x:Class="Core.Pages.Customers.MyCustomersPage">
    <ContentPage.ToolbarItems>
        <ToolbarItem Text="Info"
                     Command="{Binding ShowInfoCommand}"
                     Order="Primary"
                     Icon="{StaticResource Image.Toolbar.Info}" />
        <views:BadgeToolbarItem Text="Warnings"
                                Command="{Binding ShowWarningsCommand}"
                                Order="Primary"
                                Icon="{StaticResource Image.Toolbar.Warnings}"
                                BadgeValue="{Binding NumberOfWarnings}" />
    </ContentPage.ToolbarItems>
```

And the last but not least - here is the `UIBadgeLabel`:

```csharp
using System;
using System.Globalization;
using Cirrious.FluentLayouts.Touch;
using UIKit;

namespace iOS.Controls
{
    public sealed class UIBadgeLabel : UILabel
    {
        private const double RadiusMultiplier = 0.6;

        public UIBadgeLabel(int badgeValue) : this(badgeValue, UIFont.SmallSystemFontSize - 2)
        {
        }

        public UIBadgeLabel(int badgeValue, nfloat fontSize)
        {
            Text = badgeValue.ToString(CultureInfo.InvariantCulture);
            TextAlignment = UITextAlignment.Center;
            BackgroundColor = UIColor.Red;
            TextColor = UIColor.White;
            Font = UIFont.SystemFontOfSize(fontSize);
            ClipsToBounds = true;

            Layer.CornerRadius = (nfloat)(fontSize * RadiusMultiplier);
            Layer.BorderColor = UIColor.White.CGColor;
            Layer.BorderWidth = 1;

            this.AddConstraints(
                this.Width().EqualTo((nfloat) (fontSize * RadiusMultiplier * 2)),
                this.Height().EqualTo((nfloat) (fontSize * RadiusMultiplier * 2)));
        }
    }
}
```

Again, all the positioning is done with `FluentLayouts`. The white border for the badge was made in purpose but feel free to adjust this and any other visual or non-visual property to your needs.

I hope you will find this solution useful to some extent. I'm aware that it's not totally universal and probably will require some changes like I mentioned in few points above. Still I believe it can be some a starting point for further work and thinking of the Android (or other platforms) implementation.

Happy coding!
