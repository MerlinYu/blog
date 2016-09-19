#activity源码理解
Activity是Android的四大组件之一，其源代码中涉及了许多内容，最重要的就是生命周期，view绘制，进程之间通信。<br>
activity的继承关系如下所示：<br>
activity->ContextThemeWrapper->ContextWrapper->Context。也就是说activity的本质上是一个context<br>
##activity启动
这部分转载于老罗博客：http://blog.csdn.net/luoshengyang/article/details/6685853
![](https://github.com//merlinyu/blog/raw/master/blog_file/android/flow_control/activity_start.jpg)<br>
Step 1. 无论是通过Launcher来启动Activity，还是通过Activity内部调用startActivity接口来启动新的Activity，<br>
都通过Binder进程间通信进入到ActivityManagerService进程中，并且调用ActivityManagerService.startActivity接口；<br>
Step 2. ActivityManagerService调用ActivityStack.startActivityMayWait来做准备要启动的Activity的相关信息；<br>
Step 3. ActivityStack通知ApplicationThread要进行Activity启动调度了，这里的ApplicationThread代表的是调用ActivityManagerService.startActivity接口的进程，对于通过点击应用程序图标的情景来说，这个进程就是Launcher了，
而对于通过在Activity内部调用startActivity的情景来说，这个进程就是这个Activity所在的进程了；<br>
Step 4. ApplicationThread不执行真正的启动操作，它通过调用ActivityManagerService.activityPaused接口进入到ActivityManagerService进程中，看看是否需要创建新的进程来启动Activity；
Step 5. 对于通过点击应用程序图标来启动Activity的情景来说，ActivityManagerService在这一步中，会调用startProcessLocked来创建一个新的进程，
而对于通过在Activity内部调用startActivity来启动新的Activity来说，这一步是不需要执行的，因为新的Activity就在原来的Activity所在的进程中进行启动；<br>
Step 6. ActivityManagerServic调用ApplicationThread.scheduleLaunchActivity接口，通知相应的进程执行启动Activity的操作；<br>
Step 7. ApplicationThread把这个启动Activity的操作转发给ActivityThread，ActivityThread通过ClassLoader导入相应的Activity类，然后把它启动起来。<br>

##ActivityThread
每个App都会拥有一个各自的ActivityThread来管理app进程中的各种事务。<br>
1. ActivityThread main<br>
ActivityThread中定义了一个static main函数，这里是app启动的入口，其他的操作都是在此之后进行的。<br>
```java
public static void main(String[] args) {
    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
    SamplingProfilerIntegration.start();

    // CloseGuard defaults to true and can be quite spammy.  We
    // disable it here, but selectively enable it later (via
    // StrictMode) on debug builds, but using DropBox, not logs.
    CloseGuard.setEnabled(false);

    Environment.initForCurrentUser();

    // Set the reporter for event logging in libcore
    EventLogger.setReporter(new EventLoggingReporter());

    AndroidKeyStoreProvider.install();

    // Make sure TrustedCertificateStore looks in the right place for CA certificates
    final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
    TrustedCertificateStore.setDefaultUserDirectory(configDir);

    Process.setArgV0("<pre-initialized>");

    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    thread.attach(false);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
…………
}
```
在main函数中会进行Looper的初始化，attach绑定初始化。上层ActivityManagerService调用，启动app。<br>
2. ApplicationThread<br>
在ActivityThread中定义了ApplicationThread。ActivityThread ,ApplicationThread, Activity的对应关系是：1：1：n;当要启动Activity时
ApplicationThread发送消息（start,stop,resume...）给Handler，在Handler中今次处理这些消息，最后会调用到相应的Activity.OnCrease,Activity.OnStart...<br>
下面介绍一些关键代码的流程：
* prformlauncherActivity流程
```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
…………
// 加载类
Activity activity = null;
try {
    java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
    activity = mInstrumentation.newActivity(
            cl, component.getClassName(), r.intent);
    StrictMode.incrementExpectedActivityCount(activity.getClass());
    r.intent.setExtrasClassLoader(cl);
    r.intent.prepareToEnterProcess();
    if (r.state != null) {
        r.state.setClassLoader(cl);
    }
} catch (Exception e) {
    if (!mInstrumentation.onException(activity, e)) {
        throw new RuntimeException(
            "Unable to instantiate activity " + component
            + ": " + e.toString(), e);
    }
}
// 初始化
activity.attach(appContext, this, getInstrumentation(), r.token,
        r.ident, app, r.intent, r.activityInfo, title, r.parent,
        r.embeddedID, r.lastNonConfigurationInstances, config,
        r.referrer, r.voiceInteractor);
…………
//调用oncreate
activity.mCalled = false;
if (r.isPersistable()) {
    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
} else {
    mInstrumentation.callActivityOnCreate(activity, r.state);
}
…………
}
```
* Application onCreate<br>
handleBindApplication时会将进程名写入到系统中，并对Application进行OnCreate操作。<br>
```java
private void handleBindApplication(AppBindData data) {
// send up app name; do this *before* waiting for debugger
Process.setArgV0(data.processName);
android.ddm.DdmHandleAppName.setAppName(data.processName,
                                        UserHandle.myUserId());

try {
    mInstrumentation.callApplicationOnCreate(app);
} catch (Exception e) {
    if (!mInstrumentation.onException(app, e)) {
        throw new RuntimeException(
            "Unable to create application " + app.getClass().getName()
            + ": " + e.toString(), e);
    }
}
```
* Activity onResume<br>
onCreate,onStart 都是Activity的准备工作，在onResume中Activity的界面才会显示出来.<br>
onResume 调用过程：<br>
```java
public final ActivityClientRecord performResumeActivity(IBinder token,
r.activity.performResume();
}
Activity perfromResume
final void performResume() {
    performRestart();

    mFragments.execPendingActions();

    mLastNonConfigurationInstances = null;

    mCalled = false;
    // mResumed is set by the instrumentation
    mInstrumentation.callActivityOnResume(this);
}
ActivityThread handleResumeActivity中调用performResumeActivity，之后addview,再使activity make visible,最后通过binder通知ActivityManagerService.
final void handleResumeActivity(IBinder token,
        boolean clearHide, boolean isForward, boolean reallyResume) {
    // If we are getting ready to gc after going to the background, well
    // we are back active so skip it.
    unscheduleGcIdler();
    mSomeActivitiesChanged = true;

    // TODO Push resumeArgs into the activity for consideration
    ActivityClientRecord r = performResumeActivity(token, clearHide);

…………
if (a.mVisibleFromClient) {
    a.mWindowAdded = true;
    wm.addView(decor, l);
}

if (r.activity.mVisibleFromClient) {
    r.activity.makeVisible();
}
ActivityManagerNative.getDefault().activityResumed(token);
…………
}
```
* Activity onStop<br>
下面以stop流程为例来说明其消息传递流程。<br>
```java
// ApplicationThread
public final void scheduleStopActivity(IBinder token, boolean showWindow,
        int configChanges) {
   sendMessage(
        showWindow ? H.STOP_ACTIVITY_SHOW : H.STOP_ACTIVITY_HIDE,
        token, 0, configChanges);
}
// H Handler
public void handleMessage(Message msg) {
case STOP_ACTIVITY_SHOW:
    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStop");
    handleStopActivity((IBinder)msg.obj, true, msg.arg2);
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    break;
}
// handleStopActivity
private void handleStopActivity(IBinder token, boolean show, int configChanges) {
    ActivityClientRecord r = mActivities.get(token);
    r.activity.mConfigChangeFlags |= configChanges;

    StopInfo info = new StopInfo();
    performStopActivityInner(r, info, show, true);
………
}
// performStopActivityInner
private void performStopActivityInner(ActivityClientRecord r,
     StopInfo info, boolean keepShown, boolean saveState) {
r.activity.performStop();

…….
}
// activity
public void performStop() {
  ...
  onStop();
  ...
}
```

###binder<br>
在android中进程之间的通信是通过binder进行的，在ActivityThread中定义了一个binder的map,用来与其他进程通信：<br>
```java
final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
```
mActivities中key = IBinder，value = ActivityClientRecord。<br>
ActivityClientRecord是一个ActivityThread内部静态类，我认为可以把它当作activity的客户端记录类，
主要功能是定义了一些属性如IBinder,Window,Activity,Bundle....<br>
其中ActivityClientRecord的定义如下：<br>
```java
static final class ActivityClientRecord {
    IBinder token;
    int ident;
    Intent intent;
    String referrer;
    IVoiceInteractor voiceInteractor;
    Bundle state;
    PersistableBundle persistentState;
    Activity activity;
    Window window;
    Activity parent;
    …
}
```
