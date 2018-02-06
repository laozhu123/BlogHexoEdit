---
title: Android自动化测试
date: 2016-06-01 23:47:44
categories: "Android基础学习"
tags:
     - Android
     - 测试
     - 技术
---


![enter image description here](http://or9mw8j7a.bkt.clouddn.com/3-141209112525.jpg)



-----------------

    

### 摘要
    Android自带了很多方便的测试工具和方法，包括我们常用的单元测试、UI测试、Monkey测试、Robotium测试、MonkeyRunner测试、senevent模拟等。这些API对于我们编写高质量的APP十分有用。一方面可以发现一些隐藏问题，另一方面可以使测试过程规范化。综合以上原因，本文将分别针对Monkey测试、单元测试以及UI测试进行介绍。

### Monkey测试

#### 简介
	Monkey是Android SDK提供的一个命令行工具，可以简单、方便地运行在任何版本的Android模拟器和实体设备上。 Monkey会发送伪随机的用户事件流（如：点击、滑动、按键等，事件类别随机，就和一只猴子在试用你的APP一样，目的只为玩坏它），主要应用于APP的压力和可靠性测试。  

#### 使用方式
	（1） Monkey程序由Android系统自带，使用Java语言写成，在Android文件系统中的存放路径是： /system/framework/monkey.jar；   
	（2） Monkey.jar程序是由一个名为“monkey”的Shell脚本来启动执行，shell脚本在Android文件系统中 的存放路径是：/system/bin/monkey；  
	（3）Monkey 命令启动方式：  
	  
	    - 可以通过PC机CMD窗口中执行: adb shell monkey ｛+命令参数｝来进行Monkey测试  
	    - 或在Android机或者模拟器上直接执行monkey 命令，可以在Android机上安装Android终端模拟器  
	    - 一般使用如下命令：adb shell -p xxx.xxx.com -v 1000 进行测试，其中xxx.xxx.com是要测试的APP的包名         
	
#### 效果展示
部分输出数据如下所示：
![输出数据](http://or9mw8j7a.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170629170549.png)

#### 更多参数介绍
[点击查看](http://blog.csdn.net/linghu_java/article/details/6732895)
		
#### 可能会遇到的问题
	（1）“'adb' 不是内部或外部命令，也不是可运行的程序或批处理文件。”  
[点击查看解决方案](http://www.cnblogs.com/dwf07223/p/3228047.html)


### 单元测试
	
#### 简介
	单元测试是为了测试某一个代码单元而写的测试代码。“一个代码单元”一般就是一个方法（函数）。总结一下，我们可以这样理解：单元测试，是为了测试某一个类的某一个方法能否正常工作，而写的测试代码。Java单元测试框架：Junit、Mockito、Powermockito等,最开始建议先学习Junit & Mockito。这两款框架是java领域应用非常普及，使用简单，网上文章非常多，官网的说明也很清晰。junit运行在jvm上，所以只能测试纯java，若要测试依赖android库的代码，可以用mockito隔离依赖（下面会谈及）。

#### 使用方式
	首先我们的项目要依赖于junit库，Android studio创建项目时会自动引入该库，即在app的build.gradle中的如下语句：
	
	dependencies {
	    testCompile 'junit:junit:4.12'
    }

而后在test文件下写单元测试类
![enter image description here](http://or9mw8j7a.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170629180104.png)

被测试类如下

	public class Calculator {
	    public static int add(int a, int b) {
	        return a + b;
	    }
	}

单元测试类如下

	public class ExampleUnitTest {
	    @Test
	    public void addition_isCorrect() throws Exception {
	        assertEquals(4, Calculator.add(2,2));
	    }
	}

最后运行单元测试类，结果如下：

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170629181932.png)
	
#### Junit标签解析
在Junit中有多种标签可供使用，以下是它们的使用时机，以及作用：

	@Test： 将方法（函数）标记为测试用例
	@Before： 每一个使用@Test标记的方法运行之前都要运行一次
	@After： 每一个使用@Test标记的方法运行之后都要运行一次
	@BeforeClass： 整个测试类运行过程中，最先运行，且只运行一次
	@AfterClass： 整个测试类运行过程中，最后运行，且只运行一次

以如下代码为例：
	
	public class ExampleUnitTest {

	    @Test
	    public void addition_isCorrect() throws Exception {
	        System.out.println("@Test");
	    }
	
	
	    @Test
	    public void addition_isErr() throws Exception {
	        System.out.println("@Test");
	    }
	
	    @Before
	    public void before() throws Exception {
	        System.out.println("@Before");
	    }
	
	    @After
	    public void after() throws Exception {
	        System.out.println("@After");
	    }
	
	
	    @AfterClass
	    public static void afterClass() throws Exception {
	        System.out.println("@AfterClass");
	    }
	
	    @BeforeClass
	    public static void beforeClass() throws Exception {
	        System.out.println("@BeforeClass");
	    }
	}
	
相应的执行顺序如下：
![enter image description here](http://or9mw8j7a.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170629184021.png)

#### Mockito使用方式

简介：
		
	Mockito 是一个流行 mock 框架（mock 是指类或者接口的模拟实现，你可以自定义一个对象中某个方法的输出结果），可以和JUnit结合起来使用。Mockito 允许你创建和配置 mock 对象，并且定义它的行为。使用Mockito可以明显的简化对外部依赖的测试类的开发。

**先体验以下Mockito的使用：**

1.添加依赖

	testCompile 'org.mockito:mockito-core:2.8.47'

2.被依赖类
	
	public interface IMathUtils {
	    public int abs(int num); // 求绝对值
	}
	
3.依赖类

	@RunWith(MockitoJUnitRunner.class)
	public class MockTest {
	    @Mock
	    IMathUtils mathUtils;
	
	    @Test
	    public void mockTest() {
	
	        when(mathUtils.abs(-1)).thenReturn(1); // 当调用abs(-1)时，返回1
	
	        int abs = mathUtils.abs(-1); // 输出结果 1
	
	        Assert.assertEquals(abs, 1);// 测试通过
	    }
	}

可以发现**IMathUtils**是一个接口，根本就没有实现，用**Mockito**框架mock之后，IMathUtils.abs(-1)就有返回值1了。这就是Mockito神奇的地方！**Mockito代理了IMathUtils.abs(num)的行为**，只要调用时符合指定参数（代码中指定参数-1），就可以得到映射的返回值。

Mockito的语法**when...thenReturn...**相当直观，直观解释就是当调用某个过程时，返回固定的结果。

上述的依赖类也可以使用如下方式来写：

	public class MockTest {

        @Mock
        IMathUtils iMathUtils ; 

        @Rule public MockitoRule mockitoRule = MockitoJUnit.rule(); 

        @Test
        public void mockTest()  {
            when(mathUtils.abs(-1)).thenReturn(1); // 当调用abs(-1)时，返回1
	
	        int abs = mathUtils.abs(-1); // 输出结果 1
	
	        Assert.assertEquals(abs, 1);// 测试通过
        }
	}

其中**@Rule public MockitoRule mockitoRule = MockitoJUnit.rule();** 用于初始化Mock对象，效果与在类前添加**@RunWith(MockitoJUnitRunner.class)**标签类似

**Mock配置**

Mock有多种配置方式，如下所示：
		
	@Test
	public void test1()  {
	        //  创建 mock
	        MyClass test = Mockito.mock(MyClass.class);

        // 自定义 getUniqueId() 的返回值
        when(test.getUniqueId()).thenReturn(43);

        // 在测试中使用mock对象
        assertEquals(test.getUniqueId(), 43);
	}
	
	// 返回多个值
	@Test
	public void testMoreThanOneReturnValue()  {
	        Iterator i= mock(Iterator.class);
	        when(i.next()).thenReturn("Mockito").thenReturn("rocks");
	        String result=i.next()+" "+i.next();
	        // 断言
	        assertEquals("Mockito rocks", result);
	}
	
	// 如何根据输入来返回值
	@Test
	public void testReturnValueDependentOnMethodParameter()  {
	        Comparable c= mock(Comparable.class);
	        when(c.compareTo("Mockito")).thenReturn(1);
	        when(c.compareTo("Eclipse")).thenReturn(2);
	        // 断言
	        assertEquals(1,c.compareTo("Mockito"));
	}
	
	// 如何让返回值不依赖于输入
	@Test
	public void testReturnValueInDependentOnMethodParameter()  {
	        Comparable c= mock(Comparable.class);
	        when(c.compareTo(anyInt())).thenReturn(-1);
	        // 断言
	        assertEquals(-1 ,c.compareTo(9));
	}
	
	// 根据参数类型来返回值
	@Test
	public void testReturnValueInDependentOnMethodParameter()  {
	        Comparable c= mock(Comparable.class);
	        when(c.compareTo(isA(Todo.class))).thenReturn(0);
	        // 断言
	        Todo todo = new Todo(5);
	        assertEquals(todo ,c.compareTo(new Todo(1)));
	}
	
更多配置可以看下这个网站  [点击链接](http://static.javadoc.io/org.mockito/mockito-core/2.8.47/org/mockito/Mockito.html)


### UI测试

#### 简介
	UI测试顾名思义就是：开发人员可以对已经安装到手机或模拟器上的APP进行功能性的测试。现在Android studio自带的Espresso就是一个很好的UI测试框架。

#### 使用方式
1.配置Espresso依赖，现在Android Studio都会在项目创建时自动导入。

	testCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
	testCompile 'com.android.support.test:runner:0.4.1'

2.在androidTest目录下创建测试类

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170630135600.png)

3.被测试类（即activity之类的展示界面）
MainActivity.class
	
	public class MainActivity extends AppCompatActivity implements View.OnClickListener{

	    private EditText mEt;
	    private TextView mTv;
	    private Button mBtn;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        mEt = (EditText) findViewById(R.id.et);
	        mTv = (TextView) findViewById(R.id.tv);
	        mBtn = (Button) findViewById(R.id.btn);
	
	        mBtn.setOnClickListener(this);
	    }
	
	    @Override
	    public void onClick(View v) {
	        mTv.setText(mEt.getText().toString());
	    }
	}

activity_main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context="xgn.com.androidautotest.MainActivity">

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="15dp"
        android:padding="10dp"
        android:text="helo" />

    <EditText
        android:id="@+id/et"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"
        android:layout_marginTop="75dp" />

    <Button
        android:id="@+id/btn"
        android:layout_width="80dp"
        android:layout_height="wrap_content"
        android:layout_below="@+id/et"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="49dp"
        android:text="sure" />
	</RelativeLayout>

测试类
ExampleInstrumentedTest.class

	@RunWith(AndroidJUnit4.class)
	public class ExampleInstrumentedTest {
	    @Rule
	    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(
	            MainActivity.class);

	    @Test
	    public void useAppContext() throws Exception {
	        // Context of the app under test.
	        onView(withId(R.id.et)).perform(typeText("helo world"),
	                closeSoftKeyboard());
	        onView(withId(R.id.btn)).perform(click());
	    }
	}
其中**onView(withId(R.id.et)).perform(typeText("helo world"), closeSoftKeyboard());**选择界面中的输入框，并输入“helo world”，**onView(withId(R.id.btn)).perform(click());**选择界面中的按钮并点击。

[更多操作方式-1](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-11/Android-Espresso%E6%B5%8B%E8%AF%95%E6%A1%86%E6%9E%B6%E4%BB%8B%E7%BB%8D.md)        
[更多操作方式-2](http://blog.csdn.net/eclipsexys/article/details/45622813)

#### 标签解析

	@Rule: 应用于成员变量
	@ClassRule: 应用于测试类中的静态变量
	两者共同点：这些变量必须是TestRule接口的实例，且访问修饰符必须为public。
		
	@RunWith（AndroidJUnit4.class）：  用于修改测试运行器


### 总结
	总的来说自动化测试能够很大程度上减少开发者在测试APP上所消耗的事件，一个好的测试用例能够使APP更可靠，也可以使开发者对自己的作品更有信心。各种测试方式能够搭配使用，合理的配合能够使测试的效果达到最大化。
Android自动化测试

enter image description here

Android自动化测试
摘要
Monkey测试
简介
使用方式
效果展示
更多参数介绍
可能会遇到的问题
单元测试
简介
使用方式
Junit标签解析
Mockito使用方式
UI测试
简介
使用方式
标签解析
总结
摘要

Android自带了很多方便的测试工具和方法，包括我们常用的单元测试、UI测试、Monkey测试、Robotium测试、MonkeyRunner测试、senevent模拟等。这些API对于我们编写高质量的APP十分有用。一方面可以发现一些隐藏问题，另一方面可以使测试过程规范化。综合以上原因，本文将分别针对Monkey测试、单元测试以及UI测试进行介绍。
Monkey测试

简介

Monkey是Android SDK提供的一个命令行工具，可以简单、方便地运行在任何版本的Android模拟器和实体设备上。 Monkey会发送伪随机的用户事件流（如：点击、滑动、按键等，事件类别随机，就和一只猴子在试用你的APP一样，目的只为玩坏它），主要应用于APP的压力和可靠性测试。  
使用方式

（1） Monkey程序由Android系统自带，使用Java语言写成，在Android文件系统中的存放路径是： /system/framework/monkey.jar；   
（2） Monkey.jar程序是由一个名为“monkey”的Shell脚本来启动执行，shell脚本在Android文件系统中 的存放路径是：/system/bin/monkey；  
（3）Monkey 命令启动方式：  

    - 可以通过PC机CMD窗口中执行: adb shell monkey ｛+命令参数｝来进行Monkey测试  
    - 或在Android机或者模拟器上直接执行monkey 命令，可以在Android机上安装Android终端模拟器  
    - 一般使用如下命令：adb shell -p xxx.xxx.com -v 1000 进行测试，其中xxx.xxx.com是要测试的APP的包名         
效果展示

部分输出数据如下所示：

输出数据

更多参数介绍

点击查看

可能会遇到的问题

（1）“'adb' 不是内部或外部命令，也不是可运行的程序或批处理文件。”  
点击查看解决方案

单元测试

简介

单元测试是为了测试某一个代码单元而写的测试代码。“一个代码单元”一般就是一个方法（函数）。总结一下，我们可以这样理解：单元测试，是为了测试某一个类的某一个方法能否正常工作，而写的测试代码。Java单元测试框架：Junit、Mockito、Powermockito等,最开始建议先学习Junit & Mockito。这两款框架是java领域应用非常普及，使用简单，网上文章非常多，官网的说明也很清晰。junit运行在jvm上，所以只能测试纯java，若要测试依赖android库的代码，可以用mockito隔离依赖（下面会谈及）。
使用方式

首先我们的项目要依赖于junit库，Android studio创建项目时会自动引入该库，即在app的build.gradle中的如下语句：

dependencies {
    testCompile 'junit:junit:4.12'
}
而后在test文件下写单元测试类

enter image description here

被测试类如下

public class Calculator {
    public static int add(int a, int b) {
        return a + b;
    }
}
单元测试类如下

public class ExampleUnitTest {
    @Test
    public void addition_isCorrect() throws Exception {
        assertEquals(4, Calculator.add(2,2));
    }
}
最后运行单元测试类，结果如下：

enter image description here

Junit标签解析

在Junit中有多种标签可供使用，以下是它们的使用时机，以及作用：

@Test： 将方法（函数）标记为测试用例
@Before： 每一个使用@Test标记的方法运行之前都要运行一次
@After： 每一个使用@Test标记的方法运行之后都要运行一次
@BeforeClass： 整个测试类运行过程中，最先运行，且只运行一次
@AfterClass： 整个测试类运行过程中，最后运行，且只运行一次
以如下代码为例：

public class ExampleUnitTest {

    @Test
    public void addition_isCorrect() throws Exception {
        System.out.println("@Test");
    }


    @Test
    public void addition_isErr() throws Exception {
        System.out.println("@Test");
    }

    @Before
    public void before() throws Exception {
        System.out.println("@Before");
    }

    @After
    public void after() throws Exception {
        System.out.println("@After");
    }


    @AfterClass
    public static void afterClass() throws Exception {
        System.out.println("@AfterClass");
    }

    @BeforeClass
    public static void beforeClass() throws Exception {
        System.out.println("@BeforeClass");
    }
}
相应的执行顺序如下：

enter image description here

Mockito使用方式

简介：

Mockito 是一个流行 mock 框架（mock 是指类或者接口的模拟实现，你可以自定义一个对象中某个方法的输出结果），可以和JUnit结合起来使用。Mockito 允许你创建和配置 mock 对象，并且定义它的行为。使用Mockito可以明显的简化对外部依赖的测试类的开发。
先体验以下Mockito的使用：

1.添加依赖

testCompile 'org.mockito:mockito-core:2.8.47'
2.被依赖类

public interface IMathUtils {
    public int abs(int num); // 求绝对值
}
3.依赖类

@RunWith(MockitoJUnitRunner.class)
public class MockTest {
    @Mock
    IMathUtils mathUtils;

    @Test
    public void mockTest() {

        when(mathUtils.abs(-1)).thenReturn(1); // 当调用abs(-1)时，返回1

        int abs = mathUtils.abs(-1); // 输出结果 1

        Assert.assertEquals(abs, 1);// 测试通过
    }
}
可以发现IMathUtils是一个接口，根本就没有实现，用Mockito框架mock之后，IMathUtils.abs(-1)就有返回值1了。这就是Mockito神奇的地方！Mockito代理了IMathUtils.abs(num)的行为，只要调用时符合指定参数（代码中指定参数-1），就可以得到映射的返回值。

Mockito的语法when…thenReturn…相当直观，直观解释就是当调用某个过程时，返回固定的结果。

上述的依赖类也可以使用如下方式来写：

public class MockTest {

    @Mock
    IMathUtils iMathUtils ; 

    @Rule public MockitoRule mockitoRule = MockitoJUnit.rule(); 

    @Test
    public void mockTest()  {
        when(mathUtils.abs(-1)).thenReturn(1); // 当调用abs(-1)时，返回1

        int abs = mathUtils.abs(-1); // 输出结果 1

        Assert.assertEquals(abs, 1);// 测试通过
    }
}
其中@Rule public MockitoRule mockitoRule = MockitoJUnit.rule(); 用于初始化Mock对象，效果与在类前添加@RunWith(MockitoJUnitRunner.class)标签类似

Mock配置

Mock有多种配置方式，如下所示：

@Test
public void test1()  {
        //  创建 mock
        MyClass test = Mockito.mock(MyClass.class);

    // 自定义 getUniqueId() 的返回值
    when(test.getUniqueId()).thenReturn(43);

    // 在测试中使用mock对象
    assertEquals(test.getUniqueId(), 43);
}

// 返回多个值
@Test
public void testMoreThanOneReturnValue()  {
        Iterator i= mock(Iterator.class);
        when(i.next()).thenReturn("Mockito").thenReturn("rocks");
        String result=i.next()+" "+i.next();
        // 断言
        assertEquals("Mockito rocks", result);
}

// 如何根据输入来返回值
@Test
public void testReturnValueDependentOnMethodParameter()  {
        Comparable c= mock(Comparable.class);
        when(c.compareTo("Mockito")).thenReturn(1);
        when(c.compareTo("Eclipse")).thenReturn(2);
        // 断言
        assertEquals(1,c.compareTo("Mockito"));
}

// 如何让返回值不依赖于输入
@Test
public void testReturnValueInDependentOnMethodParameter()  {
        Comparable c= mock(Comparable.class);
        when(c.compareTo(anyInt())).thenReturn(-1);
        // 断言
        assertEquals(-1 ,c.compareTo(9));
}

// 根据参数类型来返回值
@Test
public void testReturnValueInDependentOnMethodParameter()  {
        Comparable c= mock(Comparable.class);
        when(c.compareTo(isA(Todo.class))).thenReturn(0);
        // 断言
        Todo todo = new Todo(5);
        assertEquals(todo ,c.compareTo(new Todo(1)));
}
更多配置可以看下这个网站 点击链接

UI测试

简介

UI测试顾名思义就是：开发人员可以对已经安装到手机或模拟器上的APP进行功能性的测试。现在Android studio自带的Espresso就是一个很好的UI测试框架。
使用方式

1.配置Espresso依赖，现在Android Studio都会在项目创建时自动导入。

testCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
testCompile 'com.android.support.test:runner:0.4.1'
2.在androidTest目录下创建测试类

enter image description here

3.被测试类（即activity之类的展示界面） 
MainActivity.class

public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    private EditText mEt;
    private TextView mTv;
    private Button mBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mEt = (EditText) findViewById(R.id.et);
        mTv = (TextView) findViewById(R.id.tv);
        mBtn = (Button) findViewById(R.id.btn);

        mBtn.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        mTv.setText(mEt.getText().toString());
    }
}
activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="xgn.com.androidautotest.MainActivity">

<TextView
    android:id="@+id/tv"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="15dp"
    android:padding="10dp"
    android:text="helo" />

<EditText
    android:id="@+id/et"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    android:layout_alignParentLeft="true"
    android:layout_alignParentStart="true"
    android:layout_alignParentTop="true"
    android:layout_marginTop="75dp" />

<Button
    android:id="@+id/btn"
    android:layout_width="80dp"
    android:layout_height="wrap_content"
    android:layout_below="@+id/et"
    android:layout_centerHorizontal="true"
    android:layout_marginTop="49dp"
    android:text="sure" />
</RelativeLayout>
测试类 
ExampleInstrumentedTest.class

@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(
            MainActivity.class);

    @Test
    public void useAppContext() throws Exception {
        // Context of the app under test.
        onView(withId(R.id.et)).perform(typeText("helo world"),
                closeSoftKeyboard());
        onView(withId(R.id.btn)).perform(click());
    }
}
其中onView(withId(R.id.et)).perform(typeText(“helo world”), closeSoftKeyboard());选择界面中的输入框，并输入“helo world”，onView(withId(R.id.btn)).perform(click());选择界面中的按钮并点击。

更多操作方式-1 
更多操作方式-2

标签解析

@Rule: 应用于成员变量
@ClassRule: 应用于测试类中的静态变量
两者共同点：这些变量必须是TestRule接口的实例，且访问修饰符必须为public。

@RunWith（AndroidJUnit4.class）：  用于修改测试运行器
总结

总的来说自动化测试能够很大程度上减少开发者在测试APP上所消耗的事件，一个好的测试用例能够使APP更可靠，也可以使开发者对自己的作品更有信心。各种测试方式能够搭配使用，合理的配合能够使测试的效果达到最大化。
     7539 
qqq2830
 退出账号
当前文档
 恢复至上次同步状态
 删除文档
 导出...
 预览文档
 分享链接
系统
 设置
 使用说明
 快捷帮助
 常见问题
 关于

搜索文件
杂物 Android自动化测试 
杂物 SwipeToLoadLayout 
检查Evernote中的笔记版本

