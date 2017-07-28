---
layout: post
title: Xamarin.Android - Failure INSTALL_PARSE_FAILED_MANIFEST_MALFORMED
date: 2017-04-11 19:25 +0200
tags: [xamarin, android]
categories: [mobile]
---

An error `Failure INSTALL_PARSE_FAILED_MANIFEST_MALFORMED` appears from [time to time](http://stackoverflow.com/questions/37066617/failure-install-parse-failed-manifest-malformed) in world of Android Java developers. However among Xamarin devs it seems to appear much rarely. Today I came across one and the standard Android solution did not work for me. Curious about the solution I found ([with a little help of my friends](https://www.youtube.com/watch?v=nCrlyX6XbTU))?<!-- more -->

First of all, here is the full error that is reported by tooling during the app deployment:

```
Deploying package to 'emulator-5554'
Detecting installed packages
Installing shared runtime
Installing platform framework
Installing application on device

Deployment failed because of an internal error: Unexpected install output: 	pkg: /data/local/tmp/solidbrain.space.droid-Signed.apk
Failure [INSTALL_PARSE_FAILED_MANIFEST_MALFORMED]

Deployment failed. Internal error.
```

My first try was, obviously, looking into the `AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="somenamespace.app.droid" android:installLocation="auto" android:versionCode="1" android:versionName="1.0">
	<uses-sdk android:minSdkVersion="19" android:targetSdkVersion="25" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	<application android:label="app" android:icon="@drawable/Android_App_Icon"></application>
</manifest>
```

As you can see everything seems to be fine with it. I've got a clue from my colleague that the tooling modifies the manifest file during the packaging. So the resulting signed app package might contain a `AndroidManifest.xml` file with a quite different content.

Second clue came from the [SO question](http://stackoverflow.com/questions/37066617/failure-install-parse-failed-manifest-malformed) mentioned before. The answers was focused around the casing of namespaces in the manifest.

Third piece of information were the changes I made that day in the project. In order to fix some random errors that was appearing in my app from time to time I added missing links between Java and Mono world (native services or activities). Those were constructors with `IntPtr` native object handle and registration attributes (helpful in debugging and troubleshooting) like this one:

```csharp
[Register("SomeSpace.App.Droid.Views.LoginView")]
public class SomeActivity : AppCompatActivity { }
```

Did you already spot the issue? Yup, the namespaces should be lower-case (apart from the type part):

```csharp
[Register("somespace.app.droid.views.LoginView")]
public class SomeActivity : AppCompatActivity { }
```

Apparently the packing tooling includes those types registration within the `AndroidManifest.xml`.
