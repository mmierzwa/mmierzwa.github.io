---
layout: post
title: Why ADAL does not persist user credentials on iOS simulator?
date: 2017-03-11 18:00:00 +0200
tags: [xamarin, ios]
categories: [mobile]
---

If you are using Azure Active Directory services you probably at least considered using [ADAL](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-libraries) as a client library in you application. It's easy to setup, use and it offers a unified API across the most popular platforms - iOS, Android, UWP, web - both .Net and native. Unfortunately sometimes things just does not work out of the box without deeper understanding how some features are implemented. This post is about one of them - credentials cache persistency on iOS.<!-- more -->

Running my Xamarin.iOS application on simulator I quickly noticed that I must login every time the app starts. This was an obvious sign that previously acquired tokens were not properly persisted as they should. On the other hand on the device everything was working fine. In some point this started to be irritating so I dug.

For saving the users credentials (in form of oAuth access tokens and refresh tokens) ADAL tries to use the built-in platform secure stores. The exact implementation is platform-specific and it's placed in `TokenCachePlugin` ([here](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/dev/src/ADAL.PCL.iOS/TokenCachePlugin.cs) is the implementation for iOS).

The secure store for iOS (and Mac as well) is KeyChain. So what is the difference between device and simulator? For Xamarin.iOS apps built with Xamarin Studio or Visual Studio in the `Debug|iPhoneSimulator` build configuration are not signed. As the [Apple documentation](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/02concepts/concepts.html#//apple_ref/doc/uid/TP30000897-CH204-SW1) states:

> From a high level perspective, Keychain Services uses an app’s code signature with its embedded entitlements to ensure that only an authorized app can access a particular keychain item. By default, only the app that created an item can access it in the future.

So, from the KeyChain perspective, the unsigned app deployed to simulator simply cannot store or retrieve the records.

The solution is fairly simple. In your iOS project settings, `iOS Bundle Signing` select `Debug|iPhoneSimulator` configuration and set `Provisioning Profile` to the same which you are using for the device (**not** the Automatic!):

<figure class="half center">
  <a href="/images/2017/03/setting-provisioning-profile.png" class="image-popup">
	 <img src="/images/2017/03/setting-provisioning-profile.png" alt="How to configure provisioning profile">
   </a>
	<figcaption>Xamarin Studio project settings - iOS Bundle Signing</figcaption>
</figure>

Happy coding!  
Marek
