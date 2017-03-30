#EventBus 源码理解
部分转载：http://www.cnblogs.com/angeldevil/p/3715934.html  http://www.tuicool.com/articles/jUvyUjB
## 第三方库EventBus
单进程通信机制，可以代替Handler，<br>
EventBus是一个Android端优化的publish/subscribe消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。比如请求网络，等网络返回时通过Handler或Broadcast通知UI，两个Fragment之间需要通过Listener通信，这些需求都可以通过EventBus实现。<br>
作为一个消息总线，有三个主要的元素：<br>
- Event：事件
- Subscriber：事件订阅者，接收特定的事件
- Publisher:事件发布者，用于通知Subscriber有事件发生

在EventBus中的观察者通常有四种订阅函数（就是某件事情发生被调用的方法）<br>
1、onEvent<br>
2、onEventMainThread<br>
3、onEventBackground<br>
4、onEventAsync<br>

* onEvent:如果使用onEvent作为订阅函数，那么该事件在哪个线程发布出来的，onEvent就会在这个线程中运行，也就是说发布事件和接收事件线程在同一个线程。使用这个方法时，在onEvent方法中不能执行耗时操作，如果执行耗时操作容易导致事件分发延迟。<br>
* onEventMainThread:如果使用onEventMainThread作为订阅函数，那么不论事件是在哪个线程中发布出来的，onEventMainThread都会在UI线程中执行，接收事件就会在UI线程中运行，这个在Android中是非常有用的，因为在Android中只能在UI线程中跟新UI，所以在onEvnetMainThread方法中是不能执行耗时操作的。<br>
* onEvnetBackground:如果使用onEventBackgrond作为订阅函数，那么如果事件是在UI线程中发布出来的，那么onEventBackground就会在子线程中运行，如果事件本来就是子线程中发布出来的，那么onEventBackground函数直接在该子线程中执行。<br>
* onEventAsync：使用这个函数作为订阅函数，那么无论事件在哪个线程发布，都会创建新的子线程在执行onEventAsync.<br>

##  EventBus源码分析

http://moonfacex.github.io/blog/android/2016/04/08/eventbus.html
