## Android的系统架构以及工作原理

### 系统架构

Android其本质就是在标准的Linux系统上增加了Java虚拟机Dalvik，并在Dalvik虚拟机上搭建了一个JAVA的application framework，所有的应用程序都是基于JAVA的application framework之上。
Android主要应用于ARM平台，但不仅限于ARM，通过编译控制，在X86、MAC等体系结构的机器上同样可以运行。<br>

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/android_structure.png)


#### 应用程序层
我们的Momoso,CloudMall以及绝大多数据应用都在这一层开发。<br>


#### 应用程序框架层也称(Framework层)

一般的手机厂商都在这一层做开发，实现定制ROM。
隐藏在每个应用后面的是一系列的服务和系统：

1. view视图，在Android中应用中我们所看到的按钮，文本框，列表，进度条都是在这一层定义的。
2. values资源管理。在CloudMall中实现了支持国际版本的语言，是因为这一层的资源管理器，提供了对非代码资源的访问。
3. 活动管理器（ActivityManagerService）ActivityManagerService可以管理应用的生命周期，包括启动，退出，异常恢复等等活动
4. 其他的一些主要管理器有：通知管理器，窗口管理器，内容提供器...


#### 系统运行库层
系统运行库主要有两个库：c程序库，android库

1. c程序库

 Android包含一些C/C++库，主要有2D/3D图形引擎，媒体库，浏览器webkit内核，Sqlite数据库。
   
2. Android 运行库
java 虚拟机Dalvik库。java最显著的特点就是java的虚拟机。编译器将.java文件编译成.class文件，java虚拟机最后加载.class文件运行，因为其特点，java的移植性和兼容性特别好。

  
#### Linux 内核层
Android 的核心系统服务依赖于 Linux 2.6 内核 ，如安全性，内存管理，进程管理， 网络协议栈和驱动模型。 
但是在Linux内核这一层，android对其进行了两部分的修改：

1. Binder进程间通信
  (IPC)：提供有效的进程间通信，虽然linux内核本身已经提供了这些功能，但Android系统很多服务都需要用到该功能，为了更有效的通信，android实现了自己的通信方式Binder。
  
2. 电源管理：为手持设备节省能耗。



### Android程序如何工作

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/android_start.png)




##### 应用程序结构

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/activity.png)

android的应用程序其实就是一堆activity组织在一起。当我们点击应用图标时，首先它会启动一个Application，然后再通过它来启动相应的Activity.
一般来说一个界面就是一个Activty。

android基础有四大级组件：activty,service,broadcast,contentprocider。<br>

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/activity_base_4.png)


它们的作用各不一样，activyt实现界面逻辑，service后后服务，broadcast实现对系统变化的监听例如android手机时间会发生变化时，会发出一个broadcast然后注册了个广播就可以做出反应，android系统状态的变化都是通过广播通知的，ContentProcider实现不同的应用之间的通信。

##### android view绘制的流程

view的绘制流程可以分为三步：测量view控件的大小，布局放置view，最后才是绘制view。<br>

![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/view_draw.png)


