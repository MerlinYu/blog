## Android的系统架构以及工作原理



### Android概况

android版本分布：
![](http://www.ihei5.com/uploads/allimg/2017/03/08/1-1F30Q15S3.jpg)

最新一份统计显示谷歌的安卓系统占比37.93%，超过了微软的Windows系统的37.91%。成为全球第一大操作系统。


### Android系统结构


Android其本质就是在标准的Linux系统上增加了Java虚拟机Dalvik，并在Dalvik虚拟机上搭建了一个JAVA的application framework，所有的应用程序都是基于JAVA的application framework之上。
Android主要应用于ARM平台，但不仅限于ARM，通过编译控制，在X86、MAC等体系结构的机器上同样可以运行。<br>
android在系统结构上可以分为四层，分别是：应用程序层，应用框架层，系统运行库层，以及Linux内核层。

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/android_structure.png)

#### 应用程序层
一般应用都在这一层开发，我们的Momoso,CloudMall就是在这一层开发的。<br>


#### 应用程序框架层也称(Framework层)

一般的手机厂商会在这一层做开发，实现定制的ROM。<br>

在这一层隐藏在每个应用后面的是一系列的服务和系统，其中包括：

1. view视图，在Android中应用中我们所看到的按钮，文本框，列表，进度条都是在这一层定义的。
2. values资源管理。在CloudMall中实现了支持国际版本的语言，是因为这一层的资源管理器，提供了对非代码资源的访问。
3. 活动管理器（ActivityManagerService）ActivityManagerService可以管理应用的生命周期，包括启动，退出，异常恢复等等活动
4. 其他的一些主要管理器有：通知管理器，窗口管理器，内容提供器...


#### 系统运行库层
系统运行库主要有两个库：c程序库，android库

1. c程序库

 有2D/3D图形引擎，媒体库，浏览器webkit内核，Sqlite数据库。
   
2. Android 运行库
Android运行库主要包括java虚拟机Dalvik库。java最显著的特点就是java的虚拟机。编译器将.java文件编译成.class文件二进制文件，可以几乎不经过任何修改直接运行在java虚拟机上，因为其特点，java的移植性特别好。

  
#### Linux 内核层
Android 的核心系统服务依赖于 Linux 2.6 内核 ，如安全性，内存管理，进程管理， 网络协议栈和驱动模型。 
但是在Linux内核这一层，android对其进行了两部分的修改：

1. Binder进程间通信
  linux内核本身已经提供了进行间通信的功能有：共享内存，管道，Socket,信号量，消息队列等等， 但是Android系统为了更有效的通信，android实现了自己的通信方式Binder。这种通信方式的结构如下:
  
![](http://hi.csdn.net/attachment/201107/19/0_13110996490rZN.gif)
上图可以看到Binder通信分为四部分：Client,Server,ServiceManager,驱动。<br>

2. 电源管理：为手持设备节省能耗。



### Android程序加载流程

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/android_start.png)



##### 应用程序结构

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/activity.png)

android的应用程序其实就是一堆activity组织在一起。当我们点击应用图标时，首先它会启动一个Application，然后再通过它来启动相应的Activity.
一般来说一个界面就是一个Activty。

android基础有四大级组件：activty,service,broadcast,contentprocider。<br>

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/activity_base_4.png)


它们的作用各不一样，activyt实现界面逻辑，service后后服务，broadcast实现对系统变化的监听例如android手机时间会发生变化时，会发出一个broadcast然后注册了个广播就可以做出反应，android系统状态的变化都是通过广播通知的，ContentProcider实现不同的应用之间的通信。

##### android界面

在上面中讲到Android是由一系列Activity组成的，在Activity中可以绘制界面,而Activty的界面是由一系列view组成的，最底层是一个ViewGroup然后在这个ViewGroup上添加各种view。


![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/home.png)
 
我们所看到的Android程序的界面，所有的界面都是基于view实现的，包括TextView,ImageView,ViewGroup等等都是View的子类。

view的绘制流程可以分为三步：测量view控件的大小，布局放置view，最后才是绘制view。<br>

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/view_draw.png)


