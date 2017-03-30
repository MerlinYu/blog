#RxJava源码理解
RxJava 基础讲解: http://blog.csdn.net/lzyzsd/article/details/41833541<br>
操作符：http://www.bubuko.com/infodetail-847631.html<br>
原理讲解：http://gank.io/post/560e15be2dca930e00da1083<br>
RxJava核心的功能就是异步，可以将UI进程和IO进程分开运行，实现代码简捷。其中丰富的操作符，可以将Observable自由进行转换，Schedulerss可以指定观察者和订阅者运行的线程。<br>
结合网上的博客对其源码做出分析。
## 使用示例
```java
Observable.just("Hello, world!")  
    .subscribe(s -> System.out.println(s)); 
```
## 观察订阅事件
Observable.subscribe(Subscriber) 的内部实现是这样的（关键代码）：
```java
public Subscription subscribe(Subscriber subscriber)
{
subscriber.onStart();
onSubscribe.call(subscriber);
return subscriber;
}
```
可以看到，subscriber() 做了3件事：<br>
调用 Subscriber.onStart() 是一个可选的准备方法。<br>
调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。在Call中会执行subscriber的onNext,onError,onCompleted函数。<br>
我们再看一个Observable定义的例子：
```java
observable = Observable.create(
       new Observable.OnSubscribe<String>() {
         @Override
         public void call(Subscriber<? super String> subscriber) {
           System.out.println("observable  send :hello world ");
           String hello = "hello world ";
           subscriber.onNext(hello);
           subscriber.onCompleted();
         }
       }
   );
```
在这个例子中可以清楚的看到以观察 - 订阅是如何进行的 call-subscriber->onnext,onCompleted。<br>
将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe().<br>
整个过程中对象间的关系如下图：<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/flow_control/rxjava.jpg)<br>
（转载：http://gank.io/post/560e15be2dca930e00da1083）
## 控制 Scheduler
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，
就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。<br>
1). Scheduler 的 API (一)<br>
在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。<br>
RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：<br>
- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。<br>
- Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。<br>
- Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，
区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，
可以避免创建不必要的线程。<br>
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。
这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。<br>
- 另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。 * subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。 * observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。
文字叙述总归难理解，上代码：
```java
   Observable.just(1, 2, 3, 4)
    .subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
    .observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer number) {
            Log.d(tag, "number:" + number);
        }
    }    
```
上面这段代码中，由于 subscribeOn(Schedulers.io()) 的指定，被创建的事件的内容 1、2、3、4 将会在 IO 线程发出；而由于 observeOn(AndroidScheculers.mainThread()) 的指定，因此 subscriber 数字的打印将发生在主线程 。事实上，这种在 subscribe() 之前写上两句 subscribeOn(Scheduler.io()) 和 observeOn(AndroidSchedulers.mainThread()) 的使用方式非常常见，它适用于多数的 『后台线程取数据，主线程显示』的程序策略。
那么Schedulers它们的原理是什么呢？我们看一下AndroidScheculers.mainThread()的定义：<br>
```java
public final class AndroidSchedulers {
       private AndroidSchedulers() {
           throw new AssertionError("No instances");
       }
       private static final Scheduler MAIN_THREAD_SCHEDULER =
               new HandlerScheduler(new Handler(Looper.getMainLooper()));
       /** A {@link Scheduler} which executes actions on the Android UI thread. */
       public static Scheduler mainThread() {
           Scheduler scheduler =
                   RxAndroidPlugins.getInstance().getSchedulersHook().getMainThreadScheduler();
           return scheduler != null ? scheduler : MAIN_THREAD_SCHEDULER;
       }
   }
```
在这段代码中我们由Looper.getMainLooper()可以看到返回的Scheduler运行在主线程中。再看一下observeOn(AndroidSchedulers.mainThread())的具体实现如下：<br>
```java
public final Observable<T> observeOn(Scheduler scheduler) {
    if (this instanceof ScalarSynchronousObservable) {
        return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
    }
    return lift(new OperatorObserveOn<T>(scheduler, false));
}
```
通过lift实现了一个运算符的转换返回一个Observable。<br>
我们先看一下这个构造函数：<br>
```java
new OperatorObserveOn<T>(scheduler, false)
public OperatorObserveOn(Scheduler scheduler, boolean delayError) {
    this.scheduler = scheduler;
    this.delayError = delayError;
}
```
这个构造器只是简单的赋值，并没有做其他的工作，那么接着看下看lift的转换过程。其关键代码如下：
```java
   Subscriber<? super T> st = hook.onLift(operator).call(o);
   st.onStart();
   onSubscribe.call(st);
```
先调用onStart,然后调用call这个call函数其实对应的是OperatorObserveOn中的call,代码如下：<br>
```java
@Override
public Subscriber<? super T> call(Subscriber<? super T> child) {
    if (scheduler instanceof ImmediateScheduler) {
        // avoid overhead, execute directly
        return child;
    } else if (scheduler instanceof TrampolineScheduler) {
        // avoid overhead, execute directly
        return child;
    } else {
        ObserveOnSubscriber<T> parent = new ObserveOnSubscriber<T>(scheduler, child, delayError);
        parent.init();
        return parent;
    }
}
```
其中主要做了一个init的操作：
```java
void init() {
    // don't want this code in the constructor because `this` can escape through the
    // setProducer call
    Subscriber<? super T> localChild = child;

    localChild.setProducer(new Producer() {

        @Override
        public void request(long n) {
            if (n > 0L) {
                BackpressureUtils.getAndAddRequest(requested, n);
                schedule();
            }
        }

    });
    localChild.add(recursiveScheduler);
    localChild.add(this);
}
```
在这个init当中我们终于找到了前边设置的Scheduler。再往下看：<br>
```java
@Override
public void onNext(final T t) {
    if (isUnsubscribed() || finished) {
        return;
    }
    if (!queue.offer(on.next(t))) {
        onError(new MissingBackpressureException());
        return;
    }
    schedule();
}
```
我们可以看到在onNtext中使用到了schedule();
```java
protected void schedule() {
    if (counter.getAndIncrement() == 0) {
        recursiveScheduler.schedule(this);
    }
}
```
真正运行的线程在schedule中体现。接着往下走我们知道前边的AndroidSchedulers.mainThread返回的是一个HanderScheduler,上边的schedule 会调用HanderSchedulers.schedule，代码如下：<br>
```java
@Override
public Subscription schedule(final Action0 action) {
    return schedule(action, 0, TimeUnit.MILLISECONDS);
}
@Override
public Subscription schedule(Action0 action, long delayTime, TimeUnit unit) {
    if (compositeSubscription.isUnsubscribed()) {
        return Subscriptions.unsubscribed();
    }

    action = RxAndroidPlugins.getInstance().getSchedulersHook().onSchedule(action);

    final ScheduledAction scheduledAction = new ScheduledAction(action);
    scheduledAction.addParent(compositeSubscription);
    compositeSubscription.add(scheduledAction);

    handler.postDelayed(scheduledAction, unit.toMillis(delayTime));

    scheduledAction.add(Subscriptions.create(new Action0() {
        @Override
        public void call() {
            handler.removeCallbacks(scheduledAction);
        }
    }));

    return scheduledAction;
}
```
这就是最终线程运行的代码。说的简单一些就是在Android.mainThread 返回一个由主线程的Looper创建的handler，然后在schedule时使用这个handler执行动作。
其他的线程类型Scheduler.IO等等就不作分析了。

## 操作符
常用操作符介绍，其实很多内容都是来自于ReactiveX的官方网站，英文比较好的朋友可以参考(http://reactivex.io/)。<br>
按照官方的分类，操作符大致分为以下几种：<br>
参考博客：http://www.bubuko.com/infodetail-847631.html<br>

- Creating Observables(Observable的创建操作符)，比如：Observable.create()、Observable.just()、Observable.from()等等；
- Transforming Observables(Observable的转换操作符)，比如：observable.map()、observable.flatMap()、observable.buffer()等等；
- Filtering Observables(Observable的过滤操作符)，比如：observable.filter()、observable.sample()、observable.take()等等；
- Combining Observables(Observable的组合操作符)，比如：observable.join()、observable.merge()、observable.combineLatest()等等；
- Error Handling Operators(Observable的错误处理操作符)，比如:observable.onErrorResumeNext()、observable.retry()等等；
- Observable Utility Operators(Observable的功能性操作符)，比如：observable.subscribeOn()、observable.observeOn()、observable.delay()等等；
- Conditional and Boolean Operators(Observable的条件操作符)，比如：observable.amb()、observable.contains()、observable.skipUntil()等等；
- Mathematical and Aggregate Operators(Observable数学运算及聚合操作符)，比如：observable.count()、observable.reduce()、observable.concat()等等；
- 其他如observable.toList()、observable.connect()、observable.publish()等等；

在源码中操作符都通过一个lift函数实现，返回一个新的Observable。原码如下：
```java
/**
 * Lifts a function to the current Observable and returns a new Observable that when subscribed to will pass
 * the values of the current Observable through the Operator function.
 * <p>
 * In other words, this allows chaining Observers together on an Observable for acting on the values within
 * the Observable.
 * <p> {@code
 * observable.map(...).filter(...).take(5).lift(new OperatorA()).lift(new OperatorB(...)).subscribe()
 * }
 * <p>
 * If the operator you are creating is designed to act on the individual items emitted by a source
 * Observable, use {@code lift}. If your operator is designed to transform the source Observable as a whole
 * (for instance, by applying a particular set of existing RxJava operators to it) use {@link #compose}.
 * <dl>
 *  <dt><b>Scheduler:</b></dt>
 *  <dd>{@code lift} does not operate by default on a particular {@link Scheduler}.</dd>
 * </dl>
 *
 * @param operator the Operator that implements the Observable-operating function to be applied to the source
 *             Observable
 * @return an Observable that is the result of applying the lifted Operator to the source Observable
 * @see <a href="https://github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators">RxJava wiki: Implementing Your Own Operators</a>
 */
public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
    return new Observable<R>(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber<? super R> o) {
            try {
                Subscriber<? super T> st = hook.onLift(operator).call(o);
                try {
                    // new Subscriber created and being subscribed with so 'onStart' it
                    st.onStart();
                    onSubscribe.call(st);
                } catch (Throwable e) {
                    // localized capture of errors rather than it skipping all operators
                    // and ending up in the try/catch of the subscribe method which then
                    // prevents onErrorResumeNext and other similar approaches to error handling
                    Exceptions.throwIfFatal(e);
                    st.onError(e);
                }
            } catch (Throwable e) {
                Exceptions.throwIfFatal(e);
                // if the lift function failed all we can do is pass the error to the final Subscriber
                // as we don't have the operator available to us
                o.onError(e);
            }
        }
    });
}
```
RxJava中提供的操作符最终都是通过Lift函数转换成我们需要的功能，我们可以利用Lift自定义操作符，但是一般不需要这样做。之前写过一个项目在welcome页
页面采用的就是RxJava异常加载的方式。<br>
https://github.com/MerlinYu/structure

#Reference
这篇博客的一部分内容是转载[扔物线](http://gank.io/post/560e15be2dca930e00da1083) 的感谢他的博客让我理解的RXJava.


