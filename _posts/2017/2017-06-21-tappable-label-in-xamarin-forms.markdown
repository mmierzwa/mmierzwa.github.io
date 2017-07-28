---
layout: "post"
title: "Tappable label in Xamarin.Forms"
date: "2017-06-21 18:00 +0100"
tags: [xamarin forms]
categories: [mobile]
---

Adding tap/click handling to Xamarin.Forms Label is fairly easy. You can do it both in XAML or code behind using `GesureRecognizers` collection like it is [described in this recipe](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/gestures/tap/). Unfortunately if you plan to use this solution intensively in your app it will add quite a lot of repeatable code for setting up those handlers (especially in XAML)

In this short recipe I will show how to implement a custom control that is easy to use and potentially to extend if needed.<!-- more -->

In most cases the custom controls in Xamarin.Forms are written together with custom platform renderers. This time there is no need doing this because all required functionality is contained within the standard Forms Label. All that needs to be done is to simplify its usage by exposing additional bindable properties. Those properties can be used in XAML to wire up the Labels behavior with model commands.

Here is the C# code:

```csharp
using System.Windows.Input;
using Xamarin.Forms;

namespace ExampleTappableLabelSolution
{
    public class TappableLabel : Label
    {
        public static BindableProperty TappedCommandProperty =
            BindableProperty.Create(nameof(TappedCommand), typeof(ICommand), typeof(TappableLabel), propertyChanged: TappedCommandPropertyChanged);

        public static BindableProperty TappedCommandParameterProperty =
            BindableProperty.Create(nameof(TappedCommandParameter), typeof(object), typeof(TappableLabel), propertyChanged: TappedCommandParameterPropertyChanged);

        private readonly TapGestureRecognizer _tapGestureRecognizer = new TapGestureRecognizer();

        public TappableLabel()
        {
            GestureRecognizers.Add(_tapGestureRecognizer);
        }

        public ICommand TappedCommand
        {
            get { return (ICommand) GetValue(TappedCommandProperty); }
            set { SetValue(TappedCommandProperty, value); }
        }

        public object TappedCommandParameter
        {
            get { return GetValue(TappedCommandParameterProperty); }
            set { SetValue(TappedCommandParameterProperty, value); }
        }

        private static void TappedCommandPropertyChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var label = (TappableLabel) bindable;
            label._tapGestureRecognizer.Command = (ICommand) newValue;
        }

        private static void TappedCommandParameterPropertyChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var label = (TappableLabel) bindable;
            if (newValue != null)
                label._tapGestureRecognizer.CommandParameter = newValue;
        }
    }
}
```

In order to use it in XAML first add the proper namespace declaration (assuming the custom control is in the assembly `ExampleTappableLabelSolution`):

```xml
<ContentPage ...
             xmlns:views="clr-namespace:ExampleTappableLabelSolution;assembly=ExampleTappableLabelSolution"
```

The final usage may look like this:

```xml
<views:TappableLabel Text="{Tap me}"
                     TappedCommand="{Binding MyModelTapCommand}" />
```

Remember to add `MyModelTapCommand` command in your view model class.

The single tap gesture is one of the most popular and I find this exact implementation quite useful in my daily work. Notwithstanding this solution can be easily extended to handle other types of gestures or parametrize number of taps required to fire the command. This can be done by adding new bindable properties and logic in respective `BindingPropertyChangedDelegate` methods.

Happy coding!
