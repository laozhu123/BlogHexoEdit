---
title: Dagger2的好处
date: 2016-06-01 23:47:44
categories: "Android三方框架学习"
tags:
     - Android
     - 三方框架
     - 技术
---
[TOC]


### Dagger2的好处

- 依赖的注入和配置独立于组件之外。
- 因为对象是在一个独立、不耦合的地方初始化，所以当注入抽象方法的时候，我们只需要修改对象的实现方法，而不用大改代码库。
- 依赖可以注入到一个组件中：我们可以注入这些依赖的模拟实现，这样使得测试更加简单。


### 相关标签

 **@Inject、@Module、@Provide、@Component**

- @Inject: 通常在需要依赖的地方使用这个注解。换句话说，你用它告诉Dagger这个类或者字段需要依赖注入。这样，Dagger就会构造一个这个类的实例并满足他们的依赖。

- @Module: Modules类里面的方法专门提供依赖，所以我们定义一个类，用@Module注解，这样Dagger在构造类的实例的时候，就知道从哪里去找到需要的依赖。modules的一个重要特征是它们设计为分区并组合在一起（比如说，在我们的app中可以有多个组成在一起的modules）。

- @Provide: 在modules中，我们定义的方法是用这个注解，以此来告诉Dagger我们想要构造对象并提供这些依赖。

- @Component: Components从根本上来说就是一个注入器，也可以说是@Inject和@Module的桥梁，它的主要作用就是连接这两个部分。 Components可以提供所有定义了的类型的实例，比如：我们必须用@Component注解一个接口然后列出所有的@Modules组成该组件，如果缺失了任何一块都会在编译的时候报错。所有的组件都可以通过它的modules知道依赖的范围。


### 标签使用方式

**1.@Inject**  

    public class User {  
  
	    ....  
	    ....  
	    //在被依赖类中使用@Inject标记该类的构造方法
	    @Inject  
	    public User() {  
	    }  
	  
	    ....  
	    ....    
	}  

--------------

    public class MainActivity extends AppCompatActivity {  
		  
	    //在依赖类中使用@Inject来注入被依赖类实例
	    @Inject  
	    User user;  
	    @Inject  
	    User user2;  
	    private TextView tv;  
	    private TextView tv2;  
	  
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.activity_main);  
	        //实例component，并通过其inject（）方法来对成员变量（通过@Inject进行表述的）进行赋值
	        DaggerActivityComponent.builder().build().inject(this);  
	        tv = ((TextView) findViewById(R.id.tv));  
	        tv2 = ((TextView) findViewById(R.id.tv2));  
	        tv.setText(user.toString());  
	        tv2.setText(user2.toString());  
	    }  
	} 

**2.@Component**
		
    @Component  
	public interface ActivityComponent {  
	    void inject(MainActivity activity);  
	}  

**3.@Provider & @Module**

    @Module  
	public class UserModule {  
	    @Provides
	    User providesUser() {  
	        return new User();  
	    }  
	}  



### 注入方式
- 构造方法注入：在类的构造方法前面注释@Inject
- 成员变量注入：在类的成员变量（非私有）前面注释@Inject
- 函数方法注入：在函数前面注释@Inject


### 不同类的关系

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E6%9C%AA%E6%A0%87%E9%A2%98-2-%E6%81%A2%E5%A4%8D%E7%9A%84.png)


### 编译后的生成文件与原文件及关系图

**原文件**
![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818105449.png)

**生成文件**
![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818105426.png)

**各文件对应关系**
![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E7%BC%96%E8%AF%91%E5%90%8E%E5%85%B3%E7%B3%BB%E5%9B%BE.png)


###  注入路径

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818112230.png)

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818111938.png)

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818112629.png)



### @Scope（用于划分作用域）

**javax包中自带的@Singleton,其class如下：**

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818113850.png)

**用户自己写的**

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818114133.png)

***可以看出除了少了@Documented标签外，用户自己写的Scope标签和@Singleton并没有本质的区别***

#### 各标签的意义

**@Retention**

Retention(保留)注解说明,这种类型的注解会被保留到那个阶段. 有三个值:
1.RetentionPolicy.SOURCE —— 这种类型的Annotations只在源代码级别保留,编译时就会被忽略
2.RetentionPolicy.CLASS —— 这种类型的Annotations编译时被保留,在class文件中存在,但JVM将会忽略
3.RetentionPolicy.RUNTIME —— 这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用.

**@Documented**

Documented 注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中. 示例6进一步演示了使用 


**@Scope**

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818114808.png)



### @Singleton的单例模式是如何起作用的（我只是个栗子）

该单例模式的前提是所使用的Component实例是同一个的情况下，而且任何自定义的Scope标签都有相同功能，具体实现如下：

![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818141827.png)


![enter image description here](http://or9mw8j7a.bkt.clouddn.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170818142004.png)


### @Qualifier的巧用（指哪打哪）

@Qualifier是用来给注解做注解的。它告诉Dagger依赖需求方 创建数据的时候使用哪个依赖提供方。

两个@Qualifier

    @Qualifier
	public @interface ThemeNight {
	
	}
	
	@Qualifier
	public @interface ThemeDay {
	}

-----------------------

    @Module
	public class ThemeModule {
	    @Provides
	    @ThemeDay
	    Theme provideDayTheme() {
	        return new Theme("day");
	    }

	    @Provides
	    @ThemeNight
	    Theme provideNightTheme() {
	        return new Theme("night");
	    }
	}

------------------

    @Component
    public interface ThemeComponent{
	    void inject(ThemeTest themeTest);
    }
    

--------------------

    public class ThemeTest {
	    @Inject
	    @ThemeDay
	    Theme dayTheme;
	    @Inject
	    @ThemeNight
	    Theme nightTheme;

	    public static void main(String[] arg) {
	        ThemeTest themeTest = new ThemeTest();
	        DaggerThemeComponent.create().inject(themeTest);
	        System.out.println(themeTest.dayTheme.themeName);
	        System.out.println(themeTest.nightTheme.themeName);
	    }
	}



