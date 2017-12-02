---
layout: "post"
title: "Enterprise patterns for configuration in Xamarin app. Part 1: Mocking external dependencies"
date: "2017-12-02 10:00"
tags: [design, SOLID, IoC]
categories: [mobile]
---

Modern mobile apps are rarely developed as offline-only. They typically communicate with backend services that feed them with data, keeps in sync with their web equivalents or allows for various external integrations. The backend part is most often developed by different teams in their own pace. The mobile part can be often developed faster thus it waits for the full integration.

Even if the backend service is ready to consume it's not always most convenient to develop being connected to it. Sometimes you just make a small UI change and want to see the result immediately. Even the fastest Internet connection would not give you the speed of loading the mocked data from memory. It's also easier to use the same approach in automated UI tests instead of depending on data changing in backend (if you don't test the integration itself).

Those are only two of many various scenarios where the proper design of application-level configuration can help in the development and testing process. Proper use of [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection), conditional compilation and few other techniques can make this design clean and maintainable. This spans from the local development to Continuous Integration and Deployment process. In this and next articles I will show you how to do this in Xamarin app.
<!-- more -->

## Basic mocking

Let's start with the first scenario that I described at the beginning. Assuming that both teams agree on the API, the service can be mocked in the app allowing to develop it independently of backend.

[Dependency inversion](https://en.wikipedia.org/wiki/Dependency_inversion_principle), one of the [SOLID principles](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)), suggests that all the functionality should be coded around abstractions, not the real implementations. In practice this means that our view model (in MVVM approach) or other consumers will operate on the service interface. The interface implementation will be typically instantiated by DI container. At this point the concept is most probably familiar to your development experience.

One of the main benefits of applying DI principle is that we can have more than one implementation of an interface we can switch. Consider this example:

```csharp
public interface IMyBackendService
{
  Task<IEnumerable<string>> GetValues();
}

public class HttpBackendService
{
  public async Task<IEnumerable<string>> GetValues()
  {
    // retrieve the values from backend using HttpClient, Refit etc.
  }
}

public class MockBackendService
{
  private static readonly IEnumerable<string> _mockedData = new[]
  {
    "value1", "value2", "value3"
  };

  public async Task<IEnumerable<string>> GetValues()
  {
    return await Task.FromResult(_mockedData);
  }
}
```

Now we have two `IMyBackendService` service implementations. First one provides the real data from the backend, using HTTP transport under the hood. Second returns the mocked data. `MockBackendService` could also retrieve the mocked values from the resource file (i.e. as JSON [embedded assembly resource](https://support.microsoft.com/en-us/help/319292/how-to-embed-and-access-resources-by-using-visual-c)) making it easier to store and update.

Sometimes it's worth to consider providing the mocked data on the [`HttpClientHandler`](https://msdn.microsoft.com/en-us/library/system.net.http.httpclienthandler)/[`HttpMessageHandler`](https://msdn.microsoft.com/en-us/library/system.net.http.httpmessagehandler) level. If you are interested how to do this for [Simple.OData](https://github.com/object/Simple.OData.Client) client I encourage you to read my previous post on this topic - ["Mocking HTTP response in Simple.OData client"]({% post_url 2017/2017-08-22-mocking-http-response-in-simple-odata-client %}).

## Dependency registration

The crucial part here is to provide proper service implementation to the consumer (typically the view model). Obviously it should not be made on the consumer side since it should not be aware of the specific implementation. Good place for this is the service dependency registration. In Xamarin apps you can use one of many DI containers available for PCL/.Net Standard and your target platforms: [SimpleInjector](https://simpleinjector.org/index.html), [TinyIoC](https://github.com/grumpydev/TinyIoC), [Unity](https://github.com/unitycontainer/unity), [Autofac](https://autofac.org/), [Ninject](http://www.ninject.org/) just to name the few most popular. If you are creating Xamarin.Forms app you're probably more familiar with built-in [DependencyService](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/dependency-service/introduction/).

The basic registering is quite similar in most of DI containers. Take a look on the following example registration code (Autofac):

```csharp
public void RegisterDependencies(Autofac.ContainerBuilder builder)
{
  #if OFFLINE_MODE
  builder.RegisterType<MockBackendService>().As<IMyBackendService>().SingleInstance();
  #else
  builder.RegisterType<HttpBackendService>().As<IMyBackendService>().SingleInstance();
  #endif
}
```

You will call something similar typically in your app's entry point: `Forms.Application`, iOS `AppDelegate` or Android `MainActivity` depending on the platform. The method contains a conditional compilation switch. If the actual build configuration contains the `OFFLINE_MODE` compilation symbol `MockBackendService` service implementation will be registered and used consequently when dependency is resolved. Otherwise the `HttpBackendService` will be chosen.

## Real life development

When I start developing a new feature that depends on the backend service I briefly discuss with backend team the contracts of the API and what kind of data the app should expect in real. Then I start from the mocked service and switch to the real one as soon as it's ready. This part can be done without the conditional compilation shown above since all the build configurations must work without the unfinished backend. However having the mocked service separated from the real one makes the switch quick and pretty easy.

After I make sure that the contract hasn't changed and/or making the necessary corrections I code the real implementation and add the conditional compilation logic (in slightly different form that I will show in the next post). I usually add the `OFFLINE_MODE` symbol to the `Debug` build configuration that I use only during the development and local testing. Other configurations, like ad-hoc and release builds doesn't have it so the app built on them connect to the real backend service.

In the next article I will show you how to deal with more such dependencies in a nice way. I will also show how to protect your sensitive mocked data from leaking outside of your development environment.

Happy coding!
