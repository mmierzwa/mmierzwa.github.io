---
layout: "post"
title: "How to setup Ninject as the default DI container in MvvmCross?"
date: "2017-05-07 22:40 +0100"
original_url: https://solidbrain.com/2017/04/24/how-to-setup-ninject-as-the-default-di-container-in-mvvmcross/
source_page_url: https://solidbrain.com/blog/
source_page_name: "Solidbrain Blog"
tags: [mvvmcross, android, ios]
categories: [mobile]
---

When you build a multi-platform application in .NET, especially for the mobile, you typically choose between two approaches. One is to code the shared UI layer commonly with Xamarin.Forms (you will still need to have some parts to be placed in platform projects, like custom renderers or providers). The second is to put the entire UI code in platform-specific projects. In this approach you can use the full power of each platform features (like fragments on Android). Both solutions allow for sharing common business logic between all the platforms. On the other hand full implementation of MVVM pattern in the second approach can be tricky and time consuming. The solution is to use 3rd party library; and here comes the MvvmCross. It covers many more areas than the pure MVVM pattern:

* cross-platform plug-ins,
* platform-specific helpers (i.e. Android recycler views with grouping or iOS table views with simplified customization API),
* IoC and messaging infrastructure.

Thanks to the consistent design, all the plugins and extensions, it can seriously speed-up building UI-intensive cross-platform projects. And it still allows you to take all the benefits provided by MVVM design pattern.

Now let’s see how to make Ninject and MvvmCross work together on your project.<!-- more --> I will describe how to set it up for PCL, Xamarin.iOS and Xamarin.Android (the general rules remain the same for other platforms).

You can implement everything from scratch, but I decided to use a helper package [MvvmCross.Adapter.Ninject](https://github.com/thefex/MvvmCross.Adapter.Ninject). To avoid adding new dependency and to be able to track dependencies in depth, I just add this code to my projects (it's only two classes in PCL and one for each platform). If you decide to add it as a NuGet package, remember to check for pre-release versions. Also, keep in mind that two platform-specific classes - `NinjectMvxIosSetup` and `NinjectMvxDroidSetup` - are missing in the package. In such case, remember to add them as I describe below.

The first thing to implement is the `IMvxIoCProvider`. This is the base type for DI container in MvvmCross and it's basically only the facade around the real container. It is required by MvvmCross as an interface between its API and Ninject kernel methods. `MvvmCross.Adapter.Ninject` comes with `NinjectMvxIoCProvider` class for this (full code available [here](https://github.com/thefex/MvvmCross.Adapter.Ninject/blob/master/MvvmCross.Adapter.Ninject/MvvmCross.Adapter.Ninject/NinjectMvxIoCProvider.cs)):

<script src="https://gist.github.com/mmierzwa/ed238f05efdce03a528afd74137b105e.js"></script>

Second type, [NinjectDependenciesProvider](https://github.com/thefex/MvvmCross.Adapter.Ninject/blob/master/MvvmCross.Adapter.Ninject/MvvmCross.Adapter.Ninject/NinjectDependenciesProvider.cs), is not necessarily required by MvvmCross but is a quite handy helper. It will help you separate the dependencies that are platform specific (`GetPlatformSpecificModules()`) from those defined in your common PCL project (`GetPortableNinjectModules()`).

<script src="https://gist.github.com/mmierzwa/7ff53a9bd99eeff09cf5e71db568c3ba.js"></script>

Next, you will need some code to plug this into the MvvmCross infrastructure on each platform. It will consist of two pieces: type derived from `NinjectDependenciesProvider` and the one derived from `Mvx...Setup`.

Providers job is to return the dependencies in form of standard `INinjectModule` collections. Those classes are quite straightforward:

<script src="https://gist.github.com/mmierzwa/d036f5096e481766163a8eb954f1c5eb.js"></script>

<script src="https://gist.github.com/mmierzwa/cf6a2c294c90310a54a5c1896ed5ebce.js"></script>

Most probably you will want to have at least one for PCL and for each of supported platforms:

<script src="https://gist.github.com/mmierzwa/8419729c70faa71d2c6dec47fa3a1e4e.js"></script>

`Mvx...Setup` classes are responsible for bootstrapping MvvmCross on each platform. Like the `NinjectDependenciesProvider` they are just an abstract base classes that ensures the actual platform setup types configures the DI container properly:

<script src="https://gist.github.com/mmierzwa/8ac3f9f9dd3ab8f361d28455e72f19ab.js"></script>

<script src="https://gist.github.com/mmierzwa/aad2a814ebeb0e22fb47cdb22ef2b154.js"></script>

Obviously, you can omit this level of inheritance and configure IoC directly in your `MvxSetup.InitializeLastChance()` method but I find this implementation more elegant. You will find those classes also in `MvvmCross.Adapter.Ninject` but for some reason, they are not present in NuGet package.

The final step is creating the platform-specific setup classes that derives from above abstract setups which boils down to implement `GetNinjectDependenciesProvider()` method:

<script src="https://gist.github.com/mmierzwa/dd8bc4270d32106b6f7edfa753b2bf69.js"></script>

<script src="https://gist.github.com/mmierzwa/6b098358de9e88d1d30b682f67217738.js"></script>

This configuration will enable you to use the standard `Mvx.Resolve<T>()` to resolve the dependencies from Ninject container. More importantly, it will do the same for other standard MvvmCross components.

Happy coding!
