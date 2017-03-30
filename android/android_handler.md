#android handler机制
转载相关博客：http://blog.csdn.net/itachi85/article/details/8035333

andriod提供了Handler 和 Looper 来满足线程间的通信。Handler先进先出原则。Looper类用来管理特定线程内对象之间的消息交换(MessageExchange)。<br>
1)Looper: 一个线程可以产生一个Looper对象，由它来管理此线程里的MessageQueue(消息队列)。<br>
2)Handler: Handler对象来与Looper沟通，以便push新消息到MessageQueue里;或者接收Looper从Message Queue取出)所送来的消息。<br>
3) Message Queue(消息队列):用来存放线程放入的消息。<br>
4)线程：UIthread 通常就是main thread，而Android启动程序时会替它建立一个MessageQueue。<br>

1. Handler创建消息<br>
每一个消息都需要被指定的Handler处理，通过Handler创建消息便可以完成此功能。<br>
Android消息机制中引入了消息池。Handler创建消息时首先查询消息池中是否有消息存在，如果有直接从消息池中取得，如果没有则重新初始化一个消息实例。使用消息池的好处是：消息不被使用时，并不作为垃圾回收，而是放入消息池，可供下次Handler创建消息时使用。消息池提高了消息对象的复用，减少系统垃圾回收的次数。消息的创建流程如图所示。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/handler/handler_create.png)<br>

2. Handler发送消息<br>
UI主线程初始化第一个Handler时会通过ThreadLocal创建一个Looper，该Looper与UI主线程一一对应。使用ThreadLocal的目的是保证每一个线程只创建唯一一个Looper。之后其他Handler初始化的时候直接获取第一个Handler创建的Looper。Looper初始化的时候会创建一个消息队列MessageQueue。至此，主线程、消息循环、消息队列之间的关系是1:1:1。
Handler、Looper、MessageQueue的初始化流程如图所示:<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/handler/handler_create.png)<br>
Hander持有对UI主线程消息队列MessageQueue和消息循环Looper的引用，子线程可以通过Handler将消息发送到UI线程的消息队列MessageQueue中。<br>

3. Handler处理消息<br>
UI主线程通过Looper循环查询消息队列UI_MQ，当发现有消息存在时会将消息从消息队列中取出。首先分析消息，通过消息的参数判断该消息对应的Handler，然后将消息分发到指定的Handler进行处理。子线程通过Handler、Looper与UI主线程通信的流程如图所示。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/handler/handler_create.png)<br>
从消息循环、消息发送和消息处理三个部分分析完Android应用程序的消息处理机制了，为了更深理解，这里我们对其中的一些要点作一个总结：
  >A. Android应用程序的消息处理机制由消息循环、消息发送和消息处理三个部分组成的。<br>
  B. Android应用程序的主线程在进入消息循环过程前，会在内部创建一个Linux管道（Pipe），这个管道的作用是使得Android应用程序主线程在消息队列为空时可以进入空闲等待状态，并且使得当应用程序的消息队列有消息需要处理时唤醒应用程序的主线程。<br>
  C. Android应用程序的主线程进入空闲等待状态的方式实际上就是在管道的读端等待管道中有新的内容可读，具体来说就是是通过Linux系统的Epoll机制中的epoll_wait函数进行的。<br>
  D. 当往Android应用程序的消息队列中加入新的消息时，会同时往管道中的写端写入内容，通过这种方式就可以唤醒正在等待消息到来的应用程序主线程。<br>
  E. 当应用程序主线程在进入空闲等待前，会认为当前线程处理空闲状态，于是就会调用那些已经注册了的IdleHandler接口，使得应用程序有机会在空闲的时候处理一些事情。<br>
