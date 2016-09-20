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

## google EventBus源码分析
google EventBus源码理解,并不是第三方库， 我们常用的是开源的第三方库。实现原理大体上一样。只不过第三方在dipatchEvent时做了处理。<br>
EventBus向外提供的接口只有三个register,unregister,post.<br>
```java
/**
 * Registers all subscriber methods on {@code object} to receive events.
 * Subscriber methods are selected and classified using this EventBus's
 * {@link SubscriberFindingStrategy}; the default strategy is the
 * {@link AnnotatedSubscriberFinder}.
 *
 * @param object  object whose subscriber methods should be registered.
 */
public void register(Object object) {
  Multimap<Class<?>, EventSubscriber> methodsInListener =
      finder.findAllSubscribers(object);
  subscribersByTypeLock.writeLock().lock();
  try {
    subscribersByType.putAll(methodsInListener);
  } finally {
    subscribersByTypeLock.writeLock().unlock();
  }
}
```
从上述代码分析，在register时，会寻找object所有的subscribe methodInListener(通过java代码的反射机制)，然后将其加入到订阅的map当中。
```java
/**
 * Unregisters all subscriber methods on a registered {@code object}.
 *
 * @param object  object whose subscriber methods should be unregistered.
 * @throws IllegalArgumentException if the object was not previously registered.
 */
public void unregister(Object object) {
  Multimap<Class<?>, EventSubscriber> methodsInListener = finder.findAllSubscribers(object);
  for (Entry<Class<?>, Collection<EventSubscriber>> entry :
        methodsInListener.asMap().entrySet()) {
    Class<?> eventType = entry.getKey();
    Collection<EventSubscriber> eventMethodsInListener = entry.getValue();

    subscribersByTypeLock.writeLock().lock();
    try {
      Set<EventSubscriber> currentSubscribers = subscribersByType.get(eventType);
      if (!currentSubscribers.containsAll(eventMethodsInListener)) {
        throw new IllegalArgumentException(
            "missing event subscriber for an annotated method. Is " + object + " registered?");
      }
      currentSubscribers.removeAll(eventMethodsInListener);
    } finally {
      subscribersByTypeLock.writeLock().unlock();
    }
  }
}
```
在unregister时会先寻找object的所有的methodListener 然后在for循环中在一个写锁中将其移出订阅的map。
```java
/**
 * Posts an event to all registered subscribers.  This method will return
 * successfully after the event has been posted to all subscribers, and
 * regardless of any exceptions thrown by subscribers.
 *
 * <p>If no subscribers have been subscribed for {@code event}'s class, and
 * {@code event} is not already a {@link DeadEvent}, it will be wrapped in a
 * DeadEvent and reposted.
 *
 * @param event  event to post.
 */
public void post(Object event) {
  Set<Class<?>> dispatchTypes = flattenHierarchy(event.getClass());

  boolean dispatched = false;
  for (Class<?> eventType : dispatchTypes) {
    subscribersByTypeLock.readLock().lock();
    try {
      Set<EventSubscriber> wrappers = subscribersByType.get(eventType);

      if (!wrappers.isEmpty()) {
        dispatched = true;
        for (EventSubscriber wrapper : wrappers) {
          enqueueEvent(event, wrapper);
        }
      }
    } finally {
      subscribersByTypeLock.readLock().unlock();
    }
  }

  if (!dispatched && !(event instanceof DeadEvent)) {
    post(new DeadEvent(this, event));
  }

  dispatchQueuedEvents();
}
在post的过程当中首先寻找Event的dispathTypes,然后在循环当中将满足subscribersByType.get(eventType); 加入到列队当中（enqueue）,最后在统一分发事件dispathQueuedEvents。
/**
 * Dispatches {@code event} to the subscriber in {@code wrapper}.  This method
 * is an appropriate override point for subclasses that wish to make
 * event delivery asynchronous.
 *
 * @param event  event to dispatch.
 * @param wrapper  wrapper that will call the subscriber.
 */
void dispatch(Object event, EventSubscriber wrapper) {
  try {
    wrapper.handleEvent(event);
  } catch (InvocationTargetException e) {
    try {
      subscriberExceptionHandler.handleException(
          e.getCause(),
          new SubscriberExceptionContext(
              this,
              event,
              wrapper.getSubscriber(),
              wrapper.getMethod()));
    } catch (Throwable t) {
      // If the exception handler throws, log it. There isn't much else to do!
      Logger.getLogger(EventBus.class.getName()).log(Level.SEVERE,
           String.format(
          "Exception %s thrown while handling exception: %s", t,
          e.getCause()),
          t);
    }
  }
}
```
总结来说:eventbus 在注册时将所有注册函数加入到一个复杂的map当中，在post Event时，通过Event在这个map当中寻找相对应的方法，加入到一个队列当中，最后分发事件。最后取消注册，将事件监听取消。在post和unregister事件时
需要用到一个读写锁来保证线程安全。
