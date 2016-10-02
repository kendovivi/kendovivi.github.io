# Android知识整理

##①　Intent 详解

#### 一、 Intent 作用
Intent 是一个将要执行的动作的抽象的描述，一般来说是作为参数来使用，由Intent来协助完成android各个组件之间的通讯。比如说调用startActivity()来启动一个activity,或者由broadcaseIntent()来传递给所有感兴趣的BroadcaseReceiver, 再或者由startService()/bindservice()来启动一个后台的service.所以可以看出来，intent主要是用来启动其他的activity 或者service，所以可以将intent理解成activity之间的粘合剂。

####二、 Intent构成  
要在不同的activity之间传递数据，就要在intent中包含相应的东西，一般来说数据中最基本的应该包括： 

1. __Action__ 用来指明要实施的动作是什么，比如说ACTION_VIEW, ACTION_EDIT等。
~~~
    具体的可以查阅android SDK-> reference中的Android.content.intent类，里面的constants中定义了所有的action。
~~~
2. __Data__ 要事实的具体的数据，一般由一个Uri变量来表示    
~~~
    例如：   
    ACTION_VIEW content://contacts/1 //显示identifier为1的联系人的信息    
    ACTION_DIAL  content://contacts/1 //给这个联系人打电话
~~~
除了Action和data这两个最基本的元素外，intent还包括一些其他的元素，

__Category（类别）__: 这个选项指定了将要执行的这个action的其他一些额外的信息，
例如 LAUNCHER_CATEGORY 表示Intent 的接受者应该在Launcher中作为顶级应用出现；
而ALTERNATIVE_CATEGORY表示当前的Intent是一系列的可选动作中的一个，这些动作可以在同一块数据上执行。
具体同样可以参考android SDK-> reference中的Android.content.intent类。

__Type（数据类型）__： 显式指定Intent的数据类型（MIME）。
一般Intent的数据类型能够根据数据本身进行判定，但是通过设置这个属性，可以强制采用显式指定的类型而不再进行推导。

__component（组件）__： 指定Intent的的目标组件的 类名称。
通常 Android会根据Intent 中包含的其它属性的信息，比如action、data/type、category进行查找，最终找到一个与之匹配的目标组件。
但是，如果 component这个属性有指定的话，将直接使用它指定的组件，而不再执行上述查找过程。指定了这个属性以后，Intent的其它所有属性都是可选的。

__extras（附加信息）__，是其它所有附加信息的集合。
使用extras可以为组件提供扩展信息，比如，如果要执行“发送电子邮件”这个动作，可以将电子邮件的标题、正文等保存在extras里，传给电子邮件发送组件。    

~~~
    下面是这些额外属性的几个例子：   
    ACTION_MAIN with category CATEGORY_HOME //用来 Launch home screen.  
    ACTION_GET_CONTENT with MIME type vnd.android.cursor.item/phone //用来列出列表中的所有人的电话号码   
~~~

综上可以看出，action、 data/type、category和extras 一起形成了一种语言，这种语言可以是android可以表达出诸如“给张三打电话”之类的短语组合。

####三、 intent的解析  
应用程序的组件为了告诉Android自己能响应、处理哪些隐式Intent请求，可以声明一个甚至多个Intent Filter。每个Intent Filter描述该组件所能响应Intent请求的能力——组件希望接收什么类型的请求行为，什么类型的请求数据。比如之前请求网页浏览器这个例子中，网页浏览器程序的Intent Filter就应该声明它所希望接收的Intent Action是WEB_SEARCH_ACTION，以及与之相关的请求数据是网页地址URI格式。如何为组件声明自己的Intent Filter? 常见的方法是在AndroidManifest.xml文件中用属性< Intent-Filter>描述组件的Intent Filter。 
  前面我们提到，隐式Intent(Explicit Intents)和Intent Filter(Implicit Intents)进行比较时的三要素是Intent的动作、数据以及类别。实际上，一个隐式Intent请求要能够传递给目标组件，必要通过这三个方面的检查。如果任何一方面不匹配，Android都不会将该隐式Intent传递给目标组件。接下来我们讲解这三方面检查的具体规则。
  
  1.动作测试
  Java代码   < intent-filter>元素中可以包括子元素< action>，比如：

~~~~java
    < intent-filter> 
        < action android:name=”com.example.project.SHOW_CURRENT” />
        < action android:name=”com.example.project.SHOW_RECENT” />
        < action android:name=”com.example.project.SHOW_PENDING” />  
    < /intent-filter>
~~~~
  
  一条< intent-filter>元素至少应该包含一个< action>，否则任何Intent请求都不能和该< intent-filter>匹配。如果Intent请求的Action和< intent-filter>中个某一条< action>匹配，那么该Intent就通过了这条< intent-filter>的动作测试。如果Intent请求或< intent-filter>中没有说明具体的Action类型，那么会出现下面两种情况。   (1) 如果< intent-filter>中没有包含任何Action类型，那么无论什么Intent请求都无法和这条< intent- filter>匹配;   (2) 反之，如果Intent请求中没有设定Action类型，那么只要< intent-filter>中包含有Action类型，这个 Intent请求就将顺利地通过< intent-filter>的行为测试。
