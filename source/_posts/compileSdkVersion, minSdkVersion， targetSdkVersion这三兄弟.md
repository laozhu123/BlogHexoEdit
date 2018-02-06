
---
title: compileSdkVersion, minSdkVersion， targetSdkVersion的关系
date: 2016-06-01 23:47:44
categories: "Android基础学习"
tags:
     - Android
---


![这里写图片描述](http://img.blog.csdn.net/20161130141617130)

**选择你自己的 compileSdkVersion, minSdkVersion, and targetSdkVersion**
-----------------------------------------------------------------------

当你将一个App发布出去之后，也许马上Google又发布了一个新的Android版本。那这是否就意味着你发布的App会出现一些版本上的问题。

当然这并不会困扰我们，因为Android是**向前兼容**的---向前兼容即旧版本能够适应新版本的应用---对Android而言就是在旧版上开发的应用当我们将手机的版本升级之后一样可以使用。这就是**compileSdkVersion**,  **minSdkVersion**,  **targetSdkVersion**这三者的由来，它们分别控制可用API的版本号，需要的API版本号和使用何种兼容模式。

**compileSdkVersion**
-----------------
通过compileSdkVersion你可以告诉Gradle使用何种SDK版本来编译你的App。当你在代码中使用了一些新的API时，你就需要使用相应新的Android SDK。

需要重点声明的是，**改变compileSdkVersion并不会改变运行时的行为，也就是说当在手机上跑的时候，使用的还是你手机上的SDK**，也就是你手机的Android版本号。当你改变compileSdkVersion时，会报出一些新的编译错误，但是你的compileSdkVersion并不会被包含在你的APK中：它仅仅用在编译期间。（虽然如此，但是你最好修复这些error---因为事出必定有因）

在此**强烈建议你使用最新的SDK进行编译**。对现有代码进行最新SDK的编译检查时，你可以获得的好处是：避免一些在新版本中不赞成使用的API，并且及时使用最新的API。

需要注意的是，当你使用 **Support Library**（兼容库）时，若要使用最新发布的兼容包，那么就必须使用最新版本的SDK进行编译。例：如果在gradle中添加compile 'com.android.support:design:23.0.1'那么相应的就需要将compileSdkVersion设置为23及以上。一般来讲，新版本的兼容库总是伴随新平台版本的发布，为新的API和特性提供兼容。

**minSdkVersion**
-------------
如果说compileSdkVersion是设置你可用的最新API，**那么minSdkVersion就是为你的app设置最低门槛**，低于这个门槛就不要装了。Google Play Store通过这个标记来决定你的机子是否可以安装相应的app。

该属性在开发过程中同样有着很重要的作用：默认情况下当你在开发的过程中，IDE可以通过这个标示来提醒你使用的API是否是在这个版本之后发布的，以此来帮助我们避免在运行时调用一些在手机的SDK中不存在的API。能够实现相同功能的是在代码中添加一些检查标示，来检查系统的版本来确定是否调用相应的API。

需要注意的是：当我们使用也许第三方的库时如: **Support Libraries**或者 **Google Play services**这些类型的库，这些库有他们自己相应的minSdkVersion，我们需要确保我们自己app中使用的minSdkVersion必须要大于等于第三方库的minSdkVersion。**当然也存在一些个别情况**，当我们相应使用一个第三方库，该库的minSdkVersion要高于我们app的minSdkVersion，在我们不改变我们app的minSdkVersion前提下，任然想要使用这个第三方库，那么我们需要做的是使用**tools:overrideLibrary** 标示，但是我们必须要进行彻底的测试。以防止意外的发生。

当我们要设定minSdkVersion时，可以到Google Play Store上查看最近7天的设备访问情况，这些就是你的潜在客户了。这其实最终是一个商业的决定，在于你是想要增加一定百分比的潜在客户量，还是使你的app有更好的用户体验和性能。

当然如果有一个API在你的app中很关键，那么这个决定的过程就变的很简单了。需要知道的是，即使是0.7%的潜在用户量，那也是一个很大的数字了，因为在Google Play Store的设备数是以十亿为单位的。

**targetSdkVersion**
----------------

这个版本号是这三者本文中最有趣的一个。**targetSdkVersion是Android提供向前兼容最主要的方式**，当targetSdkVersion不改变时，那么就不采取任何行为上的改变。

大多数由于targetSdkVersion改变而造成的行为改变都被记录在 VERSION_CODES中，所有细节都被列在每个发布的版本上，同时在API Levels table中有相应的链接进行说明。

例如，在Android6.0中讨论了如何针对过度到API 23的之后，如何对你的app进行运行时的权限分配模式。

由于一些行为上的改变对用户来说是可视的（取消了menu按钮，运行时权限，等），**更新到最新的SDK对多有的app都是有利的**。这并不以为这你必须使用所有的新特性或者盲目的提高你的targetSdkVersion而将测试抛在一边。----**请注意，在你提高你的targetSdkVersion之前一定要进行相应的测试**，这是提高软件质量所必须的，同时，你的用户会感激你的（也许没有感激，但是至少责骂会少很多）。

**Gradle and SDK versions**
-----------------------

基于以上内容让我们知道设置正确的 compileSdkVersion, minSdkVersion , targetSdkVersion是相当重要的。也许你会想，如果在Android Studio和Gradle中，这些值都相应的整合进了工具系统中，如在模块的buile.gradle文件中设置好了。如下面这样：

```
android {
  compileSdkVersion 23
  buildToolsVersion “23.0.1”

  defaultConfig {
    applicationId “com.example.checkyourtargetsdk"
    minSdkVersion 7
    targetSdkVersion 23
    versionCode 1
    versionName “1.0”
  }
}
```

在外面的两个：compileSdkVersion 和buildToolsVersion 一个是编译时SDK的版本号，一个是编译时编译器的版本号，这个我在我的领一篇文章中有详细介绍[链接地址](http://blog.csdn.net/qqq2830/article/details/53405699)。

在**defaultConfig**中的内容则是项目构建的基础设置。

在这当中 compileSdkVersion是和编译时有关，而minSdkVersion， targetSdkVersion这两者最终会放假APK中。你可以在生成的AndroidManifest.xml文件中看到：

```
<uses-sdk android:targetSdkVersion=”23" android:minSdkVersion=”7" />
```

你会发现当你手动将这些设置进你的manifest中后，如果你使用Gradle进行build，那么它会忽略你的设置。

**总的来说三者的关系**
----

```
minSdkVersion <= targetSdkVersion <= compileSdkVersion
```
**理想化来说应该是这样：**

```
minSdkVersion (lowest possible) <= 
    targetSdkVersion == compileSdkVersion (latest SDK)
```

通过这样，你就可以获得最多的潜在用户数，同时使app性能更好，同时界面更酷炫。

期待您的加入 [Google+ post](https://plus.google.com/+AndroidDevelopers/posts/4TRW8SztAHv?utm_campaign=adp_series_sdkversion_010616&utm_source=medium&utm_medium=blog) 和[ Android Development Patterns Collection](https://plus.google.com/collection/sLR0p?utm_campaign=adp_series_sdkversion_010616&utm_source=medium&utm_medium=blog)