#java 线程
进程和线程的区别：<br>
  进程：每个进程都有独立的代码和数据空间（进程的上下文），进程间的切换会有较大的开销，一个进程中会包含多个线程。
  线程：同一类线程共享数据和代码空间，每个线程都有独立运行栈和程序计数器。
  线程和进程都分为五个阶段：创建，就绪，运行，堵塞，终止。

##线程创建方式
* Thread<br>
* Runnable<br>
* 线程池(Executor)<br>
  
  ### Thread
  ```java
  thread1.start();
class Thread1 extends Thread {

  @Override
  public void run() {
    super.run();
    System.out.println("thread one is running");
    for (int i = 0; i < 100000; i++) {

    }
    System.out.println("thread one is ended");
  }
  ```
  ### Runnable
  ```java
  Thread thread = new Thread(demo.printRunnable);
thread.start();

Runnable printRunnable = new Runnable() {
  @Override
  public void run() {
    System.out.println("print.......");
  }
};
```
这两种方式一般采用Runnable的形式，Runnable 可以实现代码共享，并且不需要继承Thread。
### Executor
ThreadPoolExecutor会有callback状态返回<br>
在Android中一般使用AsyncTask来实现线程池。
##线程的调度
###线程的优先级
Java 的线程的优先级用整数表示，取值范围是1-10，Thread类有以下三个静态常量：<br>
```java
static int MAX_PRIORITY = 10 ;// 线程的最高优先级
static int MIN_PRIOPRITY = 1;// 线程的最低优先级
static int NORM_PRIOPRITY = 5;// 默认优先级
```
每个线程都有默认的优先级，主线程的默认优先级为NORM_PRIOPRITY.<br>
###线程的睡眠
```java
Thread.sleep(long mills)
```
###线程让步
Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。
###线程的加入
join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，
当前线程再由阻塞转为就绪状态。
###线程的唤醒
Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。
选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 
直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；
例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。




