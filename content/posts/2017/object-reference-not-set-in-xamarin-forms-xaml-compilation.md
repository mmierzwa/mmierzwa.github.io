---
categories:
- mobile
date: "2017-11-06T00:00:00Z"
tags:
- xamarin forms
- xaml
title: Object reference not set in Xamarin.Forms XAML compilation
---

XAML compilation is (or at least should be) one of the default optimisation steps in Xamarin.Forms app development. It really speeds up the app especially on Android. Sometimes however it can cause some nasty errors like this one:

```
/path/to/project/Views/MyContentPage.xaml : error : Object reference not set to an instance of an object
```


First time I saw it I had absolutely no clue what was the reason. The [common solutions for similar issues with VS for Mac designer](https://forums.xamarin.com/discussion/63201/xaml-compilation-object-reference-not-set-to-an-instance-of-an-object) did not help. I started with trying to narrow down the problematic code using the standard bisection method. After few cut-compile-paste cycles I knew that the root cause was this line in my XAML file:

```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <!-- ... -->
        <converters:SurveyAnswerStringConverter x:Key="SurveyAnswerStringConverter" />
    </ResourceDictionary>
</ContentPage.Resources>
```

For some reason the XAML compiler could not comprehend creation of the converter. I had quite a lot of converters in my app but this one was specific in terms of inheritance hierarchy - it's base class was abstract:

```csharp
public class SurveyAnswerStringConverter : MissingDataConverterBase<string>
{
    public override string Convert(string value, Type targetType, object parameter, CultureInfo culture)
    {
        return Convert(value, false, value, Resources.Survey_NotAnswered);
    }
}
```

```csharp
public abstract class MissingDataConverterBase<TTarget> : ValueConverterBase<string, TTarget>
{
    protected virtual TTarget Convert(string value, bool allowEmptyString, TTarget presentValue, TTarget missingValue)
    {
        if (allowEmptyString)
            return value == null ? missingValue : presentValue;

        return string.IsNullOrEmpty(value) ? missingValue : presentValue;
    }
}
```

Obviously this should not be a problem. It works pretty well without XAML compilation and does not crash in runtime. Nevertheless XAML compiler has some problem with it.

Fortunately the workaround is quite simple: instead of declaring the resource object in XAML I moved it to runtime, right after the XAML initialisation:

```csharp

public MyContentPage()
{
    InitializeComponent();

    Resources.Add("MissingDataStringConverter", new MissingDataStringConverter());
}
```

## Divide and conquer

Just a one final thought. Even if your issue is different than this one I still strongly recommend the bisection method mentioned above.

It's pretty straightforward. Cut the first half (more or less) of your XAML and compile. If it compiles - you know that the problematic code is here. Otherwise do the same with the second part. Divide this part in half and continue recursively until you find the buggy line(s).

This is exactly the same procedure as introduced in [`git bisect` tool](https://git-scm.com/docs/git-bisect) for finding a commit that introduced a particular bug and it's really effective (comparing to random guesses).

Happy coding!
