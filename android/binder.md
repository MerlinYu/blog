#Binder 理解
转载：http://blog.csdn.net/boyupeng/article/details/47011383<br>
转载：http://blog.csdn.net/coding_glacier/article/details/7520199
Binder是Android进程之间通信方式。Linux进程之间的通信包括：管道（pipe）,信号(singal)，跟踪（Trace）,插口（Socket），报文队列（Message），共享内享(Share Memory),
和信号量（Semaphore）。Binder是专门为android进程之间设计的一种通信方式，其实质上是一种CLient-Server通信模式。<br>
在Android系统中，为了向应用开发者提供丰富多样的功能，媒体播放，视音频频捕获，到各种让手机更智能的传感器（加速度，方位，温度，光亮度等）都由不同的Server负责管理，
应用程序只需做为Client与这些Server建立连接便可以使用这些服务，花很少的时间和精力就能开发出令人眩目的功能。<br>
Client-Server方式的广泛采用对进程间通信（IPC）机制是一个挑战。目前linux支持的IPC包括传统的管道，System V IPC，即消息队列/共享内存/信号量，
以及socket中只有socket支持Client-Server的通信方式。当然也可以在这些底层机制上架设一套协议来实现Client-Server通信，但这样增加了系统的复杂性，在手机这种条件复杂，资源稀缺的环境下可靠性也难以保证。
另一方面是传输性能。socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。消息队列和管道采用存储-转发方式，
即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。共享内存虽然无需拷贝，但控制复杂，难以使用。<br>
各种IPC方式数据拷贝次数

IPC  | 数据o拷贝次数
--------- | --------
共享内享  | 0 
Binder  | 1 
Socket/管道/消息队列  | 2 
还有一点是出于安全性考虑。终端用户不希望从网上下载的程序在不知情的情况下偷窥隐私数据，连接无线网络，长期操作底层设备导致电池很快耗尽等等。
传统IPC没有任何安全措施，完全依赖上层协议来确保。首先传统IPC的接收方无法获得对方进程可靠的UID和PID（用户ID进程ID），从而无法鉴别对方身份。
Android为每个安装好的应用程序分配了自己的UID，故进程的UID是鉴别进程身份的重要标志。使用传统IPC只能由用户在数据包里填入UID和PID，但这样不可靠，容易被恶意程序利用。可靠的身份标记只有由IPC机制本身在内核中添加。其次传统IPC访问接入点是开放的，无法建立私有通道。比如命名管道的名称，systemV的键值，socket的ip地址或文件名都是开放的，只要知道这些接入点的程序都可以和对端建立连接，不管怎样都无法阻止恶意程序通过猜测接收方地址获得连接。
基于以上原因，Android需要建立一套新的IPC机制来满足系统对通信方式，传输性能和安全性的要求，这就是Binder。<br>
Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送发添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。

##Binder 通信模型
Binder框架定义了四个角色：Server，Client，ServiceManager（以后简称SMgr）以及Binder驱动。其中Server，Client，SMgr运行于用户空间，驱动运行于内核空间。
这四个角色的关系和互联网类似：Server是服务器，Client是客户终端，SMgr是域名服务器（DNS），驱动是路由器。
 
 
####1. service manager
Service Manager是一个linux级的进程,顾名思义，就是service的管理器。这里的service是什么概念呢？这里的service的概念和init过程中init.rc中的service是不同，init.rc中的service是都是linux进程，但是这里的service它并不一定是一个进程，也就是说可能一个或多个service属于同一个linux进程。在这篇文章中不加特殊说明均指Android native端的service。
任何service在被使用之前，均要向SM(Service Manager)注册，同时客户端需要访问某个service时，应该首先向SM查询是否存在该服务。如果SM存在这个service，那么会将该service的handle返回给client，handle是每个service的唯一标识符。







