---
layout: "post"
title: "Enterprise patterns for configuration in Xamarin app. Part 3: Versioning and keeping secrets secret"
date: "2018-01-21 23:00"
tags: [design, SOLID, IoC]
categories: [mobile]
---
It's very common to have multiple versions of the app during development - i.e. stable beta and store/production or alpha that contains the latest changes. Managing application configuration for multiple versions might be confusing when it's not carefully designed and setup with the build process.

Publishing app to App Store or Google Play developers often forget that the mobile application is running de facto in hostile environment. Advanced user can easily reverse engineer the installed package on rooted/jailbroken device or even an emulator and see the data that wasn't supposed to be released on production. As you will see this is just a different aspect of the multi-version app config.

This article is a continuation of [previous]({% post_url 2017/2017-12-02-enterprise-patterns-for-configuration-in-xamarin-app-part-1 %}) [posts]({% post_url 2017/2017-12-06-enterprise-patterns-for-configuration-in-xamarin-app-part-2 %}). If you haven't read those I strongly recommend doing it now since I'll refer to them here.
<!-- more -->

## Environments, versions, configurations

Most of the time I'm developing Xamarin apps (and other types as well) in IDE I do it in `Debug` config. This build is usually setup to produce full PDBs with no linking. Like I wrote in [the first part]({% post_url 2017/2017-12-02-enterprise-patterns-for-configuration-in-xamarin-app-part-1 %}) of this series I usually add a compilation symbols that indicates i.e. offline app behaviour.

Nevertheless, `Debug` configuration is obviously not the best option for distribution and manual QA tests. I won't go into much detail about versioning strategy in mobile app development. This topic will be covered in the next series about continuous integration and delivery.

Typically, there is an alpha version that includes the latest changes, beta that acts as a release candidate and the store/production version. From the app configuration perspective those versions may vary in many different aspects, such as:

* backend endpoint URLs (to different environments, like test, stage or production)
* timeout values
* number of retries for the backend calls

and other, not only related to backend integration.

## Configurations in practice

To differentiate between alpha/beta/store version in build we can use the same technique which is used to distinguish code that is to be built for different platforms in shared projects - compilation symbols. In order for this to work, symbols need to be defined in `DefineConstants` element of `PropertyGroup` for each relevant configuration.

Here is an example for `Alpha` version in PCL project:

```xml
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Alpha|AnyCPU' ">
  ...
  <DefineConstants>TRACE;DEBUG;ALPHA_BUILD</DefineConstants>
  ...
</PropertyGroup>
```

Now when we have `ALPHA_BUILD`, `BETA_BUILD`, `STORE_BUILD` etc. symbols defined we can use them in code. The most straightforward way to vary the app configuration is to use conditional compilation per-setting:

```csharp
public static class ApplicationConfiguration
{
  // test backend:
  #if ALPHA_BUILD
  public static readonly Uri MyServiceUrl = new Uri("https://test.example.com/api");
  // production backend:
  #elif BETA_BUILD || STORE_BUILD
  public static readonly Uri MyServiceUrl = new Uri("https://example.com/api");
  #endif
}
```

This might the right approach for one to few such settings. But soon, when the number of settings increases you will end up with the same issues like I described in [my previous post]({% post_url 2017/2017-12-06-enterprise-patterns-for-configuration-in-xamarin-app-part-2 %}) when discussing the DI modules registration. Furthermore, using static configuration creates a strong coupling between your services and configuration itself.

In the same way as for service configuration [pictured in the previous article]({% post_url 2017/2017-12-06-enterprise-patterns-for-configuration-in-xamarin-app-part-2 %}) [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control) principle comes to rescue. You can design your app configuration in the same manner as the services:

```csharp
public interface IApplicationConfiguration
{
  Uri MyServiceUrl { get; }
  TimeSpan ServiceTimeout { get; }
  //...
}

public class TestApplicationConfiguration : IApplicationConfiguration
{
  public Uri MyServiceUrl => new Uri("https://test.example.com/api");
  public TimeSpan ServiceTimeout => TimeSpan.FromSeconds(30);
  //...
}

public class ProductionApplicationConfiguration : IApplicationConfiguration
{
  public Uri MyServiceUrl => new Uri("https://example.com/api");
  public TimeSpan ServiceTimeout => TimeSpan.FromSeconds(10);
  //...
}
```

Then the only place where you need to use conditional compilation is the dependency registration:

```csharp
#if ALPHA_BUILD
builder.RegisterType<TestApplicationConfiguration>().As<IApplicationConfiguration>().SingleInstance();
#elif BETA_BUILD || STORE_BUILD
builder.RegisterType<ProductionApplicationConfiguration>().As<IApplicationConfiguration>().SingleInstance();
#endif
```

## Security matters

The last solution has one drawback. When you release your app to the App Store or Google Play it can be downloaded and decompiled by anyone with sufficient skills and knowledge. This means that some details of your development/testing environment may leak outside your organisation. From my experience such environments are rarely the subject of careful protection in the same extent as the production (which is of course a bad practice). For this reason, it's better to prone the store version from all such unnecessary information.

This can be done in few ways. The simplest is to add the conditional compilation to configuration classes:

```csharp
#if ALPHA_BUILD
public class TestApplicationConfiguration : IApplicationConfiguration
{
  public Uri MyServiceUrl => new Uri("https://test.example.com/api");
  public TimeSpan ServiceTimeout => TimeSpan.FromSeconds(30);
  //...
}
#endif

#if BETA_BUILD || STORE_BUILD
public class ProductionApplicationConfiguration : IApplicationConfiguration
{
  public Uri MyServiceUrl => new Uri("https://example.com/api");
  public TimeSpan ServiceTimeout => TimeSpan.FromSeconds(10);
  //...
}
#endif
```

It's not very pretty but it does its job. We can improve it further.

## But what about data?

Before I show you the second approach let's think about the different problem for a while. Remember when I showed [the strategies of mocking the data for development]({% post_url 2017/2017-12-02-enterprise-patterns-for-configuration-in-xamarin-app-part-1 %})? My preferred way for doing it is adding embedded resource files into the common PCL/netstandard project and reading them on demand. Such files might potentially contain much more sensitive data than your test endpoint URLs. But JSON files cannot have conditional compilation included inside the file.

The solution for this problem, which can be also applied for regular C\# code files, is to use the conditional **inclusion** into the project file:

```xml
<ItemGroup Condition=" '$(Configuration)' == 'Debug' ">
  <EmbeddedResource Include="Data\MockData.json" />
</ItemGroup>
```

In this example `MockData.json` file will be included in build only for the `Debug` configuration, where it belongs.

## Wrapping up

As you can see creating the maintainable application configuration is a matter of using best OOP practices and basic common sense. None of the approaches I showed in this or previous articles is unfamiliar for the regular programmer. Still putting this all together requires some practice. I strongly encourage you practice using SOLID principles in your daily development. They will make your life much easier.

Happy coding!
