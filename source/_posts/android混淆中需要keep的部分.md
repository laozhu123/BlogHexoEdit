---
title: proguard
date: 2018-04-08 00:00:44
categories: "Android基础学习"
tags:
     - Android
---

#### 简介
> Proguard是一个集文件压缩,优化,混淆和校验等功能的工具
它检测并删除无用的类,变量,方法和属性
它优化字节码并删除无用的指令.
它通过将类名,变量名和方法名重命名为无意义的名称实现混淆效果.
最后它还校验处理后的代码

保留某个包下面的类以及子包

```
-keep public class com.droidyue.com.widget.**
```

保留所有类中使用otto的public方法

```
# Otto
-keepclassmembers class ** {
    @com.squareup.otto.Subscribe public *;
    @com.squareup.otto.Produce public *;
}
```

保留Contants类的BOOK_NAME属性

```
-keepclassmembers class com.example.admin.proguardsample.Constants {
     public static java.lang.String BOOK_NAME;
}
```

### -dontwarn
>dontwarn是一个和keep可以说是形影不离,尤其是处理引入的library时.

引入的library可能存在一些无法找到的引用和其他问题,在build时可能会发出警告,如果我们不进行处理,通常会导致build中止.因此为了保证build继续,我们需要使用dontwarn处理这些我们无法解决的library的警告.

比如关闭Twitter sdk的警告,我们可以这样做

```
-dontwarn com.twitter.sdk.**
```

### 哪些不应该混淆

#### 反射中使用的元素
>如果一些被混淆使用的元素(属性,方法,类,包名等)进行了混淆,可能会出现问题,如NoSuchFiledException或者NoSuchMethodException等.
>
>想要验证,我们需要看一看混淆的映射文件,文件名为mapping.txt,该文件保存着混淆前后的映射关系.

```
com.example.admin.proguardsample.Constants -> com.example.admin.proguardsample.a:
    java.lang.String BOOK_NAME -> a
    void <init>() -> <init>
    void <clinit>() -> <clinit>
com.example.admin.proguardsample.MainActivity -> com.example.admin.proguardsample.MainActivity:
    void <init>() -> <init>
    void onCreate(android.os.Bundle) -> onCreate
```

#### GSON的序列化与反序列化
>因为反序列化创建对象本质还是利用反射,会根据json字符串的key作为属性名称,value则对应属性值.

**序列化**
```
Item toSerializeItem = new Item();
toSerializeItem.id = 2;
toSerializeItem.name = "Apple";
String serializedText = gson.toJson(toSerializeItem);
Log.i(LOGTAG, "testGson serializedText=" + serializedText);
```


**反序列化**

```
Gson gson = new Gson();
Item item = gson.fromJson("{\"id\":1, \"name\":\"Orange\"}", Item.class);
Log.i(LOGTAG, "testGson item.id=" + item.id + ";item.name=" + item.name);
```

- 如何解决
>1.将序列化和反序列化的类排除混淆
2.使用@SerializedName注解字段

```
public class Item {
    @SerializedName("name")
    public String name;
    @SerializedName("id")
    public int id;
```

#### 枚举也不要混淆

枚举使用起来很简单,如下

```
public enum Day {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY
}
```
>默认的Proguard配置已经处理了枚举相关的keep操作.

```
# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

enum文件编译后产生的class文件如下：

```
➜  proguardsample javap  Day
Warning: Binary file Day contains com.example.admin.proguardsample.Day
Compiled from "Day.java"
public final class com.example.admin.proguardsample.Day extends java.lang.Enum<com.example.admin.proguardsample.Day> {
  public static final com.example.admin.proguardsample.Day MONDAY;
  public static final com.example.admin.proguardsample.Day TUESDAY;
  public static final com.example.admin.proguardsample.Day WEDNESDAY;
  public static final com.example.admin.proguardsample.Day THURSDAY;
  public static final com.example.admin.proguardsample.Day FRIDAY;
  public static final com.example.admin.proguardsample.Day SATURDAY;
  public static final com.example.admin.proguardsample.Day SUNDAY;
  public static com.example.admin.proguardsample.Day[] values();
  public static com.example.admin.proguardsample.Day valueOf(java.lang.String);
  static {};
}
```

#### 四大组件不建议混淆
>四大组件声明必须在manifest中注册,如果混淆后类名更改,而混淆后的类名没有在manifest注册,是不符合Android组件注册机制的.
外部程序可能使用组件的字符串类名,如果类名混淆,可能导致出现异常

#### 注解不能混淆
>注解在Android平台中使用的越来越多,常用的有ButterKnife和Otto.很多场景下注解被用作在运行时反射确定一些元素的特征.

>为了保证注解正常工作,我们不应该对注解进行混淆.Android工程默认的混淆配置已经包含了下面保留注解的配置

```
-keepattributes *Annotation*
```

#### 其他不该混淆的
>jni调用的java方法
java的native方法
js调用java的方法
第三方库不建议混淆
其他和反射相关的一些情况


stacktrace的恢复
Proguard混淆带来了很多好处,但是也会导致我们收集到的崩溃的stacktrace变得更加难以读懂,好在有补救的措施,这里就介绍一个工具,retrace,用来将混淆后的stacktrace还原成混淆之前的信息.

retrace脚本
Android 开发环境默认带着retrace脚本,一般情况下路径为./tools/proguard/bin/retrace.sh

mapping映射表
Proguard进行混淆之后,会生成一个映射表,文件名为mapping.txt,我们可以使用find工具在Project下查找

```
find . -name mapping.txt
./app/build/outputs/mapping/release/mapping.txt
```

一个崩溃stacktrace信息
一个原始的崩溃信息是这样的.

```
E/AndroidRuntime(24006): Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'int android.graphics.Bitmap.getWidth()' on a null object reference
E/AndroidRuntime(24006):    at com.example.admin.proguardsample.a.a(Utils.java:10)
E/AndroidRuntime(24006):    at com.example.admin.proguardsample.MainActivity.onCreate(MainActivity.java:22)
E/AndroidRuntime(24006):    at android.app.Activity.performCreate(Activity.java:6106)
E/AndroidRuntime(24006):    at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1123)
E/AndroidRuntime(24006):    at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2566)
E/AndroidRuntime(24006):    ... 10 more
```

对上面的信息处理,去掉E/AndroidRuntime(24006):这些字符串retrace才能正常工作.得到的字符串是

```
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'int android.graphics.Bitmap.getWidth()' on a null object reference
at com.example.admin.proguardsample.a.a(Utils.java:10)
at com.example.admin.proguardsample.MainActivity.onCreate(MainActivity.java:22)
at android.app.Activity.performCreate(Activity.java:6106)
at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1123)
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2566)
... 10 more
```

将上面的stacktrace保存成一个文本文件,比如名称为npe_stacktrace.txt.

开搞

```
./tools/proguard/bin/retrace.sh   /Users/admin/Downloads/ProguardSample/app/build/outputs/mapping/release/mapping.txt /tmp/npe_stacktrace.txt
```
得到的易读的stacktrace是

```
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'int android.graphics.Bitmap.getWidth()' on a null object reference
at com.example.admin.proguardsample.Utils.int getBitmapWidth(android.graphics.Bitmap)(Utils.java:10)
at com.example.admin.proguardsample.MainActivity.void onCreate(android.os.Bundle)(MainActivity.java:22)
at android.app.Activity.performCreate(Activity.java:6106)
at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1123)
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2566)
... 10 more
```

注意:为了更加容易和高效分析stacktrace,建议保留SourceFile和LineNumber属性

```
-keepattributes SourceFile,LineNumberTable
```

