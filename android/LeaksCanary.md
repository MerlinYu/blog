#android leakCanary 内存泄漏检测
参考博客：http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/
##使用
```bash
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
 }
```
在application中初始化
```java
private RefWatcher mRefWatcher;
public static void watchRef(Object obj) {
  if (null != sApp && null != sApp.mRefWatcher) {
    sApp.mRefWatcher.watch(obj);
  }
}

@Override
public void onCreate() {
  super.onCreate();
mRefWatcher = LeakCanary.install(this);
}
```
下一步将要被监测的类加入到队列中
```java
protected void onDestroy() {
IMayGou.watchRef(this);
}
```
##原理
相关博客：http://www.jianshu.com/p/0049e9b344b0<br>
http://hongjiang.info/java-referencequeue/<br>
http://vjson.com/wordpress/leakcanary源码分析第二讲－refwatcher详解.html<br>
github:https://github.com/square/leakcanary<br>

LeakCanary 的机制如下：<br>

- RefWatcher.watch() 会以监控对象来创建一个 KeyedWeakReference 弱引用对象<br>
- 在 AndroidWatchExecutor 的后台线程里，来检查弱引用已经被清除了，如果没被清除，则执行一次 GC<br>
- 如果弱引用对象仍然没有被清除，说明内存泄漏了，系统就导出 hprof 文件，保存在 app 的文件系统目录下<br>
- HeapAnalyzerService 启动一个单独的进程，使用 HeapAnalyzer 来分析 hprof 文件。它使用另外一个开源库 HAHA。<br>
- HeapAnalyzer 通过查找 KeyedWeakReference 弱引用对象来查找内在泄漏<br>
- HeapAnalyzer 计算 KeyedWeakReference 所引用对象的最短强引用路径，来分析内存泄漏，并且构建出对象引用链出来。<br>
- 内存泄漏信息送回给 DisplayLeakService，它是运行在 app 进程里的一个服务。然后在设备通知栏显示内存泄漏信息。<br>
LeakCanary 检测内存的关键代码如下:<br>
```java
/**
 * Watches the provided references and checks if it can be GCed. This method is non blocking,
 * the check is done on the {@link Executor} this {@link RefWatcher} has been constructed with.
 *
 * @param referenceName An logical identifier for the watched object.
 */
public void watch(Object watchedReference, String referenceName) {
  checkNotNull(watchedReference, "watchedReference");
  checkNotNull(referenceName, "referenceName");
  if (debuggerControl.isDebuggerAttached()) {
    return;
  }
  final long watchStartNanoTime = System.nanoTime();
  String key = UUID.randomUUID().toString();
  retainedKeys.add(key);
  final KeyedWeakReference reference =
      new KeyedWeakReference(watchedReference, key, referenceName, queue);

  watchExecutor.execute(new Runnable() {
    @Override public void run() {
      ensureGone(reference, watchStartNanoTime);
    }
  });
}

void ensureGone(KeyedWeakReference reference, long watchStartNanoTime) {
  long gcStartNanoTime = System.nanoTime();

  long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);
  removeWeaklyReachableReferences();
  if (gone(reference) || debuggerControl.isDebuggerAttached()) {
    return;
  }
  gcTrigger.runGc();
  removeWeaklyReachableReferences();
  if (!gone(reference)) {
    long startDumpHeap = System.nanoTime();
    long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);

    File heapDumpFile = heapDumper.dumpHeap();

    if (heapDumpFile == HeapDumper.NO_DUMP) {
      // Could not dump the heap, abort.
      return;
    }
    long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
    heapdumpListener.analyze(
        new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
            gcDurationMs, heapDumpDurationMs));
  }
}

private boolean gone(KeyedWeakReference reference) {
  return !retainedKeys.contains(reference.key);
}
```
LeakCanary在watch中通过执行Executor去检测引用有没有被清除。其中gone()函数就是用来判断引用有没有被清除。<br>
但是我们还有一个疑惑的地方就是这样的判断为什么能证明引用已经不存在？<br>
###引用回收
http://hongjiang.info/java-referencequeue/<br>
http://vjson.com/wordpress/leakcanary源码分析第二讲－refwatcher详解.html<br>
这两篇博客的内容是关于ReferenceQuene的知识其中有讲到引用回收的机制。<br>
###install
在Application中执行LeakCanary.install(this);<br>
```java
public static RefWatcher install(Application application,
    Class<? extends AbstractAnalysisResultService> listenerServiceClass,
    ExcludedRefs excludedRefs) {
  if (isInAnalyzerProcess(application)) {
    return RefWatcher.DISABLED;
  }
  enableDisplayLeakActivity(application);
  HeapDump.Listener heapDumpListener =
      new ServiceHeapDumpListener(application, listenerServiceClass);
  RefWatcher refWatcher = androidWatcher(application, heapDumpListener, excludedRefs);
  ActivityRefWatcher.installOnIcsPlus(application, refWatcher);
  return refWatcher;
}
```

Android 自定义了一个ActivityWatcher 在其中监测Activity的生命周期onActivityDestroyed 时执行RefWatcher.wathch(activity);<br>
Android 中有一个AndroidWatchExecutor 实现了Executor的接口，其中重写了execute方法关键代码如下：<br>
```java
@Override public void execute(final Runnable command) {
  if (isOnMainThread()) {
    executeDelayedAfterIdleUnsafe(command);
  } else {
    mainHandler.post(new Runnable() {
      @Override public void run() {
        executeDelayedAfterIdleUnsafe(command);
      }
    });
  }
}

void executeDelayedAfterIdleUnsafe(final Runnable runnable) {
  // This needs to be called from the main thread.
  Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
    @Override public boolean queueIdle() {
      backgroundHandler.postDelayed(runnable, delayMillis);
      return false;
    }
  });
}
```
这两段代码的主要功能是:延迟一段时间在后台中执行Runnable,LeaksCanary中设置延迟时间是5000ms.



