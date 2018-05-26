---
layout: "post"
title: "How to detect screen keyboard appearance changes"
date: "2018-05-26 22:00"
tags: [xamarin forms, ios, android]
categories: [mobile]
---

Detecting on-screen keyboard toggles and proper handling of such changes can be quite tricky. Android tries to deal with those events on its own but its behavior is often far from perfect. iOS, on the other hand, leaves all the work to an app developer. Both approaches have its advantages and disadvantages but sooner or later each mobile app dev will have to face this problem.

While there are [few posts](https://www.codeproject.com/Articles/1172935/Detecting-Software-Keyboard-Events-in-Xamarin-Andr) or [SO](https://stackoverflow.com/questions/4745988/how-do-i-detect-if-software-keyboard-is-visible-on-android-device) [questions](https://stackoverflow.com/questions/2150078/how-to-check-visibility-of-software-keyboard-in-android) on this matter I found no comprehensive text so far.

In this article, I'll focus on detection. I will show you how to instrument your Xamarin.Forms app so you could react on soft keyboard toggles in a unified manner both on iOS and Android.
<!-- more -->

The general idea is to have a platform component that, once started, will notify the entire app about two facts: that the software keyboard is displayed/hidden and what is its height.

I decided to use the cross-platform messaging mechanism. I use it very rare cases when I really find no better and cleaner solution. It's really easy to lose control over the app behaviour when [`MessagingCenter`](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/messaging-center) or [MvvmCross `Messenger`](https://www.mvvmcross.com/documentation/plugins/messenger) is overused. In this case, however, I found this approach clean enough.

Just one more thing before we start. In my Forms apps I use wrapper types to have strongly typed messages, so don't be surprised when you see such constructs in further code (interface `IMessenger` and messages derived from `Message` base). I don't get into much details about messaging implementation to not move away from the main topic.

Here is the message that represents keyboard toggles:

```csharp
public class KeyboardToggledMessage : Message
{
    public bool IsDisplayed { get; }
    public double KeyboardHeight { get; }

    public KeyboardToggledMessage(object sender, bool isDisplayed, double keyboardHeight)
        : base(sender)
    {
        IsDisplayed = isDisplayed;
        KeyboardHeight = keyboardHeight;
    }
}
```

And this is the keyboard toggle detector abstraction that will be implemented on each platform separately:

```csharp
public interface IKeyboardNotificationProvider
{
    void StartNotifying();
    void StopNotifying();
}
```

iOS implementation is pretty simple since the UIKit operates on exactly the same concept that is being implemented here. It just needs to be adapted to the cross-platform form:

```csharp
public class KeyboardNotificationProvider : IKeyboardNotificationProvider
{
    private static IMessenger Messenger => ServiceLocator.Messenger;

    private double _keyboardHeight;

    public void StartNotifying()
    {
        UIKeyboard.Notifications.ObserveWillShow(OnKeyboardShown);
        UIKeyboard.Notifications.ObserveWillHide(OnKeyboardHidden);
    }

    public void StopNotifying()
    {
    }

    private void OnKeyboardShown(object sender, UIKeyboardEventArgs e)
    {
        var frame = e.FrameEnd;
        _keyboardHeight = frame.Height;

        Messenger.Send(new KeyboardToggledMessage(this, true, _keyboardHeight));
    }

    private void OnKeyboardHidden(object sender, UIKeyboardEventArgs e)
    {
        Messenger.Send(new KeyboardToggledMessage(this, false, _keyboardHeight));
    }
}
```

The height of the keyboard is given in the same units as in Forms, so no additional calculations are required here.

The registration is done in `AppDelegate.FinishedLaunching()` method:

```csharp
public class AppDelegate : Xamarin.Forms.Platform.iOS.FormsApplicationDelegate
{
    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        Xamarin.Forms.Forms.Init();

        InitContainer();

        LoadApplication(new App());

        ServiceLocator.KeyboardNotificationProvider.StartNotifying();

        return base.FinishedLaunching(application, launchOptions);
    }
}
```

The Android implementation is more complex. It's based on the concept of global layout changes listeners.

There is no straightforward way of detecting if the keyboard has been displayed or hidden. Fortunately, we can make a quite confident guess about this event by checking if the system is currently expecting text input from the user.

The second obstacle is calculating the keyboard height. The implementation bellow caches and updates the height based on purely empiric constant `KeyboardDisplayedToHiddenRatio`. It is the ratio of the app screen height visible to the user (only app controls) to the height of area hidden to the user.

```csharp
public class KeyboardNotificationProvider : IKeyboardNotificationProvider
{
    private KeyboardListner _keyboardListner;

    private static ViewTreeObserver CurrentViewTreeObserver => ActivityProvider.RootContentView.ViewTreeObserver;

    public void StartNotifying()
    {
        _keyboardListner = _keyboardListner ?? new KeyboardListner();
        CurrentViewTreeObserver.AddOnGlobalLayoutListener(_keyboardListner);
    }

    public void StopNotifying()
    {
        CurrentViewTreeObserver.RemoveOnGlobalLayoutListener(_keyboardListner);
    }

    [Register("mymd.mobile.droid.services.KeyboardListner")]
    public class KeyboardListner : Java.Lang.Object, ViewTreeObserver.IOnGlobalLayoutListener
    {
        private static IMessenger Messenger => ServiceLocator.Messenger;
        private const float KeyboardDisplayedToHiddenRatio = 0.15f;

        private double _keyboardHeight;

        private static InputMethodManager _inputManager;

        public KeyboardListner()
        {
            _inputManager = GetInputManager();
        }

        public KeyboardListner(IntPtr handle, JniHandleOwnership transfer)
            : base(handle, transfer)
        {
        }

        public void OnGlobalLayout()
        {
            TryCalculateKeyboardHeight();
            NotifyOnKeyboardToggled();
        }

        private void TryCalculateKeyboardHeight()
        {
            var contentView = ActivityProvider.RootContentView;

            if (contentView == null)
                return;

            var windowVisibleDisplayFrame = new Rect();
            contentView.GetWindowVisibleDisplayFrame(windowVisibleDisplayFrame);

            var visibleScreenHeight = contentView.RootView.Height;
            var potentialKeyboardHeight = visibleScreenHeight - windowVisibleDisplayFrame.Bottom;

            if (potentialKeyboardHeight > visibleScreenHeight * KeyboardDisplayedToHiddenRatio)
                _keyboardHeight = Math.Ceiling(potentialKeyboardHeight.ToFormsScreenValue());
        }

        private void NotifyOnKeyboardToggled()
        {
            if (_inputManager.Handle == IntPtr.Zero)
                _inputManager = GetInputManager();

            if (_inputManager.IsAcceptingText && _keyboardHeight > 0)
                Messenger.Send(new KeyboardToggledMessage(this, true, _keyboardHeight));
            else
                Messenger.Send(new KeyboardToggledMessage(this, false, _keyboardHeight));
        }

        private static InputMethodManager GetInputManager()
            => (InputMethodManager) ActivityProvider.CurrentActivity.GetSystemService(Context.InputMethodService);
    }
}
```

Additionally, Android screen units are not 1:1 with the Forms units since they strictly depend on screen density. That's why we will need some additional extension class to calculate the final measure:

```csharp
public static class ControlsExtension
{
    public static DisplayMetrics DisplayMetrics => Application.Context.Resources.DisplayMetrics;
    public static float DisplayDensity => DisplayMetrics.Density;

    public static double ToFormsScreenValue(this int androidScreenValue)
        => (double) androidScreenValue / DisplayDensity;
}
```

I also use this simple helper class to obtain the current activity (regardless of the fact that Forms usually use only single activity):

```csharp
public class ActivityProvider
{
    public static Activity CurrentActivity { get; set; }

    public static View RootContentView
        => CurrentActivity.FindViewById(Android.Resource.Id.Content);
}
```

The initialisation is done in app's `MainActivity`:

```csharp
public class MainActivity : Xamarin.Forms.Platform.Android.FormsAppCompatActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        ActivityProvider.CurrentActivity = this;

        //...

        base.OnCreate(bundle);

        InitContainer();

        Forms.Init(this, bundle);

        ServiceLocator.KeyboardNotificationProvider.StartNotifying();

        LoadApplication(new App());
    }

    protected override void OnDestroy()
    {
        base.OnDestroy();
        ServiceLocator.KeyboardNotificationProvider.StopNotifying();
    }

    protected override void OnResume()
    {
        base.OnResume();
        ServiceLocator.KeyboardNotificationProvider.StartNotifying();
    }
}
```

The example consumption can be implemented using the [observer pattern](https://en.wikipedia.org/wiki/Observer_pattern):

```csharp
public class KeyboardToggledObserver : MessageObserver<KeyboardToggledMessage>
{
    private bool _previousKeyboardDisplayed;

    protected override void OnMessageArrived(KeyboardToggledMessage message)
    {
        if (_previousKeyboardDisplayed != message.IsDisplayed)
        {
            Debug.WriteLine($"[{nameof(KeyboardToggledObserver)}] Keboard toggled: displayed={message.IsDisplayed}, height={message.KeyboardHeight}");
            _previousKeyboardDisplayed = !_previousKeyboardDisplayed;
        }
    }
}
```

Again, forgive me that I'm not giving the full messaging implementation here. I hope you will get the main point and implement this detail in a way that fits you the best. Let me know - preferably in comments below this post - if you would want to see it on this blog. I will write a separate article.

Happy coding!
