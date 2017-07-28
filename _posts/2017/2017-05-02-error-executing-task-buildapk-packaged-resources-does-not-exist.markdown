---
layout: post
title: "Error executing task BuildApk: packaged_resources does not exist"
date: 2017-05-02 07:00 +0100
tags: [xamarin, android]
categories: [mobile]
---

Have you ever encountered an error: `Error executing task BuildApk: .../bin/packaged_resources does not exist`? If so you probably know that solving this issue can be sometimes quite tricky. It may be hard to track when and where the true bug was introduced in the code base. Although some suggestions can be [found on the Xamarin Forum](https://forums.xamarin.com/discussion/63356/the-file-obj-debug-android-bin-packaged-resources-does-not-exist) the solution usually differ case by case.<!-- more -->

The initial error may look like this:

```
Build FAILED.
Errors:

/.../MyApp.csproj (SignAndroidPackage) ->
/Library/Frameworks/Mono.framework/External/xbuild/Xamarin/Android/Xamarin.Android.Common.targets (_BuildApkEmbed target) ->

/Library/Frameworks/Mono.framework/External/xbuild/Xamarin/Android/Xamarin.Android.Common.targets: error : Error executing task BuildApk: obj/Debug/android/bin/packaged_resources does not exist
```

It doesn't tell to much. The true issue is usually related not with the `BuildApk` task itself but with the previous one - generating the app package resources with `AAPT` tool.

To track down the true error first turn on the detailed build logs. [Here is a great instruction](https://forums.xamarin.com/discussion/27515/how-to-obtain-diagnostic-build-logs) how to do this step by step and where to find the log output.

Next step is to find the error in the detailed log. Look for the `error APT` string. Here is an example:

Final:

```
Executing package -f -m -M obj/Debug/android/manifest/AndroidManifest.xml -J /var/folders/ls/dtrrhlz12qb0cqwvqhs008b80000gn/T/zct7dmic.ctk --custom-package my.app.droid -F obj/Debug/android/bin/packaged_resources.bk -S obj/Debug/res/ -S
...
Resources/layout/RoomDetailView.axml(2): error APT0000: No resource identifier found for attribute 'compatElevation' in package 'my.app.droid'
Done executing task "Aapt"
```

In my case the error was caused by wrongly defined axml attribute `local:compatElevation`:

```xml
<android.support.design.widget.FloatingActionButton
	                    android:id="@+id/roomDetailBookRoomButton"
	                    android:layout_width="wrap_content"
	                    android:layout_height="wrap_content"
	                    local:srcCompat="@drawable/ic_lock_outline_white_24px"
	                    android:clickable="true"
	                    local:MvxBind="Click BookRoomCommand; Visibility RoomInfo, Converter=BookButtonVisibility"
	                    local:backgroundTint="@color/mint"
	                    android:elevation="15dp"
	                    local:compatElevation="15dp"
						android:layout_centerInParent="true"/>
```

Like I wrote before - it's usually different in each case but now you know how to check what is the issue with your app.

Happy coding!
