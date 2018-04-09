---
title: 点击图标启动activity的过程
date: 2018-02-26 12:02:30
categories: "Android源码学习"
tags:
     - android
     - 源码
     - 技术
---

![enter image description here](http://hi.csdn.net/attachment/201108/14/0_1313305334OkCc.gif)

**1.在ActivityStack.startActivityLocked（创建ActivityRecord）**

**2.在ActivityStack.startActivityUncheckedLocked（创建TaskRecord）**

3.startActivityLocked -->  
startActivityUncheckedLocked --> 
startActivityLocked --> 
resumeTopActivityLocked --> 
startPausingLocked 
**-binder->** 
**ApplicationThreadProxy.schedulePauseActivity -->** 

**（下面的两行是message的传递）**
ActivityThread.queueOrSendMessage --> 
H.handleMessage --> 

**ActivityThread.handlePauseActivity（真正执行pause）** 

**-binder->** 
**ActivityManagerProxy.activityPaused -->** 
**ActivityStack.activityPaused（在stack中做一些pause之后的处理） -->**
ActivityStack.completePauseLocked --> 

ActivityStack.resumeTopActivityLocked(launcer已停止，需启动mainactivity) -->  

ActivityStack.startSpecificActivityLocked --> 
**ActivityManagerService.startProcessLocked(开始创建进程，并将ActivityThread添入其中)**

ProcessRecord app = getProcessRecordLocked(processName, info.uid);  
int pid = Process.start("android.app.ActivityThread",  
                mSimpleProcessManagement ? app.processName : null, uid, uid,  
                gids, debugFlags, null); 

**ActivityThread.main（启动ActivityThread）** 
**-binder->** 
ActivityManagerProxy.attachApplication 

通过pid将processRecord取回，放在app变量中，然后对app的其它成员进行初始化，最后调用mMainStack.realStartActivityLocked执行真正的Activity启动操作

**ActivityStack.realStartActivityLocked** 
**-binder->** 
**ApplicationThreadProxy.scheduleLaunchActivity -->** 

ActivityThread.queueOrSendMessage --> 
**ActivityThread.handleLaunchActivity（真正launcher new Activity）** 

这里首先调用performLaunchActivity函数来加载这个Activity类，即shy.luo.activity.MainActivity，然后调用它的onCreate函数，最后回到handleLaunchActivity函数时，再调用handleResumeActivity函数来使这个Activity进入Resumed状态，即会调用这个Activity的onResume函数，这是遵循Activity的生命周期的。

**ActivityThread.performLaunchActivity -->**

**1函数前面是收集要启动的Activity的相关信息，主要package和component信息**
**2然后通过ClassLoader将shy.luo.activity.MainActivity类加载进来**
**3接下来是创建Application对象，这是根据AndroidManifest.xml配置文件中的Application标签的信息来创建的**
**4后面的代码主要创建Activity的上下文信息，并通过attach方法将这些上下文信息设置到MainActivity中去**
**5最后还要调用MainActivity的onCreate函数**

这里不是直接调用MainActivity的onCreate函数，而是通过mInstrumentation的callActivityOnCreate函数来间接调用，前面我们说过，mInstrumentation在这里的作用是监控Activity与系统的交互操作，相当于是系统运行日志。



 一. Step1 - Step 11：Launcher通过Binder进程间通信机制通知ActivityManagerService，它要启动一个Activity；

二. Step 12 - Step 16：ActivityManagerService通过Binder进程间通信机制通知Launcher进入Paused状态；

三. Step 17 - Step 24：Launcher通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就创建一个新的进程，用来启动一个ActivityThread实例，即将要启动的Activity就是在这个ActivityThread实例中运行；

四. Step 25 - Step 27：ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给ActivityManagerService，以便以后ActivityManagerService能够通过这个Binder对象和它进行通信；

五. Step 28 - Step 35：ActivityManagerService通过Binder进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。 