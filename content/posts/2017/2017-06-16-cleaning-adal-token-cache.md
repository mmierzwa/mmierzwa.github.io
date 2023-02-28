---
categories:
- mobile
date: "2017-06-16T00:00:00Z"
original_url: https://solidbrain.com/2017/05/22/cleaning-adal-token-cache-on-android-and-ios/
source_page_name: Solidbrain Blog
source_page_url: https://solidbrain.com/blog/
tags:
- adal
- android
- ios
title: Cleaning ADAL token cache on Android and iOS
---

Microsoft [Azure Active Directory Authentication Libraries](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-libraries) (ADAL) is a popular set wrapper around [Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/) API distributed in the form of platform and language specific components. It's especially useful in multi-platform applications that integrate with various AD APIs such as Outlook or Graph API. It not only wraps the oAuth endpoints but automates the entire application flow for retrieving, refreshing and persisting tokens.

Unfortunately, among many features, ADAL does not provide the logout functionality out of the box. Let's see how to implement this in few simple steps.<!--more-->

First one is to define a common abstraction that can be referenced in PCL:

<script src="https://gist.github.com/mmierzwa/08d60d39692557d9f939d8cec365bd8b.js"></script>

Next is the implementation of Android provider. On this platform [ADAL stores](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/dev/src/Microsoft.IdentityModel.Clients.ActiveDirectory/Platforms/android/TokenCachePlugin.cs) tokens in [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html) under specific name and key. With this knowledge the implementation is straightforward:

<script src="https://gist.github.com/mmierzwa/3692b6cc9d7a26c167e661879e61786a.js"></script>

iOS implementation is quite similar. It uses the [KeyChain](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/02concepts/concepts.html#//apple_ref/doc/uid/TP30000897-CH204-SW8) for the [same purpose](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/dev/src/Microsoft.IdentityModel.Clients.ActiveDirectory/Platforms/iOS/TokenCachePlugin.cs) and the removal is pretty simple:

<script src="https://gist.github.com/mmierzwa/3a2057b6a8c906d836c347960796b8b4.js"></script>

At the end let's consider a sample scenario - clearing user credentials (tokens) in [MVVMCross](https://www.mvvmcross.com) application on reinstallation. To achieve this the app needs to remember if it has been run since the last uninstall or if it's the first time. This state can be persisted using i.e. the [Xamarin.Settings](https://github.com/jamesmontemagno/SettingsPlugin) plugin (in the example it's wrapped with `IApplicationSettings` interface). In the first case, it would also be nice to not bother the user showing him the login screen.

Here's the sample code:

<script src="https://gist.github.com/mmierzwa/a11c66cbfd13caf1aa65908c58f22ee1.js"></script>

Happy coding!
