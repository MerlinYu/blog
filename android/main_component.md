#android四大组件基本知识
##Activity
* 继承关系：activity->ContextThemeWrapper->ContextWrapper->Context<br>
* 启动方式：显示启动和隐式启动<br>
* 加载方式：standard， singleTop， singleTask， singleInstance，默认是standard方式加载Activity.
```java
<activity android:name="ActB" android:launchMode="singleTask"></activity>
```
四种加载模式说明<br>

1. standard ：系统的默认模式，一次跳转即会生成一个新的实例。假设有一个activity命名为Act1，执行语句：<br>
startActivity(new Intent(Act1.this, Act1.class));<br>
后Act1将跳转到另外一个Act1，也就是现在的栈里面有 Act1 的两个实例。按返回键后你会发现仍然是在Act1（第一个）里面。<br>

2. singleTop：singleTop 跟standard 模式比较类似。唯一的区别就是，当跳转的对象是位于栈顶的activity（应该可以理解为用户眼前所 看到的activity）时，程序将不会生成一个新的activity实例，而是直接跳到现存于栈顶的那个activity实例。如果不位于栈顶则创建新的实例。<br>
3. singleTask： singleTask模式和后面的singleInstance模式都是只创建一个实例的。在这种模式下，无论跳转的对象是不是位于栈顶的activity，程序都不会生成一个新的实例（当然前提是栈里面已经有这个实例）。在跳转的时候会将栈中位于此activity上的其他activity移除掉。当startActivity时不会重新oncreate 会进入到onNewIntent 中加载 数据intent.<br>
4. singleInstance: 设置为 singleInstance 模式的 activity 将独占一个task（相当于另一个进程），独占一个task的activity与其说是activity，倒不如说是一个应用，这个应用与其他activity是独立的，它有自己的上下文activity。<br>

* 生命周期：<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/activity_life.png)<br>

## Service
* 启动方式:startservice 和bindservice.<br>

1. startService 启动的服务主要用于启动一个服务执行后台任务，不进行通信。停止服务使用stopService。<br>
startService的流程：<br>
context.startService->ActivityManagerService->…->ActivityThread.ApplicationThread.scheduleCreateService->queueOrSendMessage.sendMessage->Hadler.handleMessage.handleCreateService->service.oncreate->...<br>
startService的生命周期：<br>
启动时，startService –> onCreate() –> onStart()停止时，stopService –> onDestroy()如果调用者直接退出而没有停止Service，
则Service 会一直在后台运行 Context.startService()方法启动服务，在服务未被创建时，系统会先调用服务的onCreate()方法，接着调用onStart()方法。
如果调用startService()方法前服务已经被创建，多次调用startService()方法并不会导致多次创建服务，但会导致多次调用onStart()方法。<br>

2. bindService 启动的服务该方法启动的服务要进行通信。停止服务使用unbindService。<br>
bindService()方式的生命周期：<br>
绑定时,bindService -> onCreate() –> onBind()调用者退出了，即解绑定时,Srevice就会unbindService –>onUnbind() –> onDestory()Context.bindService()方式启动 Service的方法：
绑定Service需要三个参数：bindService(intent, conn, Service.BIND_AUTO_CREATE);第一个：Intent对象第二个：ServiceConnection对象，创建该对象要实现它的onServiceConnected()和 onServiceDisconnected()来判断连接成功或者是断开连接第三个：
如何创建Service，一般指定绑定的时候自动创建。<br>

* 继承关系：Service->ContextWrapper->Context。

## BroadCast
* 注册方式：静态注册和动态注册
* 生命周期：
broadcastReceiver的生命周期在onReceive return时结束，同时指明如果需要show dialog or notify 需要启动startService来进行。
BroadCast中不能处理耗时事件。BroadCastReceiver中onReceive运行的时间不超过10s，如果long-running可以在onReceive中启动service。<br>
* 注册广播：
注册的流程很琐碎但是实质上是：将广播接收器receiver及其要接收的广播类型filter保存在ActivityManagerService中，以后进行相应的广播处理。
* 发送广播：

1. 通过sendBroadcast把一个广播通过Binder进程间通信机制发送给ActivityManagerService，ActivityManagerService根据这个广播的Action类型找到相应的广播接收器，然后把这个广播放进自己的消息队列中去，就完成第一阶段对这个广播的异步分发了；
2. ActivityManagerService在消息循环中处理这个广播，并通过Binder进程间通信机制把这个广播分发给注册的广播接收分发器ReceiverDispatcher，ReceiverDispatcher把这个广播放进MainActivity所在的线程的消息队列中去，就完成第二阶段对这个广播的异步分发了；
3. ReceiverDispatcher的内部类Args在MainActivity所在的线程消息循环中处理这个广播，最终是将这个广播分发给所注册的BroadcastReceiver实例的onReceive函数进行处理。<br>

## ContentProvider
