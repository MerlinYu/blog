#android bug 总结一
##UI bug总结
1. RecyclerView view holder内容显示错乱<br>
原因：RecyclerView Holder 复用，添加绑定view时如果有addview操作需要先removeallview。<br>
2. activity 打开照相机之后，状态改变<br>
原因：activity打开照相机activity被销毁，返回activity时重新Oncreate恢复activity.<br>
解决办法：1.onSaveInstanceState时保存activy数据;2.在Mainfest.xml文件设置该activity的属性：<br>
android:configChanges = "ketboardHidden|orientation|screenSize"表示屏幕横竖屏，键盘切换时该activity的状态不会改变。<br>
3. Picasso load本地图片不成功。<br>
原因：Picasso在load图片时地址形式是URI的形式。从本地获得的图片地址如果是：storges/../...jpg必须转换。<br>
解决办法：String str = Uri.formFile(new File(path)).toString。
4. handler sendmessage 手机黑屏<br>
原因：在OnCreate 中使用handler.sendmessage会导致手机黑屏<br>
5. android 软键盘弹出造成View空白<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/ui/keyboard.png)<br>
当我们打开Activity输入时如果有EditText控件时会自动弹出软键盘以便输入文字。<br>
在工作中发现这样一种情况：B Activity含有EditText,在onResume中有网络请求，从A Activity跳转到B Activity中再返回A发现A Activity view弹出软键盘界面空白。<br>
这种情况发生的原因我猜想是：当跳转到B Activity时要访问网络，耗时初始化，异步弹出软键盘，因为B activity view还没有初始化完成，这个软键盘会在A Activity上也显示，当B activity初始化完成，又会显示在B Activity，再次返回A activity时会造成view空白。<br>
解决办法：B activity不主动弹出软键盘。<br>
```java
android:windowSoftInputMode="stateHidden|adjustPan"
``` 
6. android webview 加载h5界面部分图片没有加载出来<br>
webview 使用的是chrome 内核。
查看log显示：
"Mixed Content: The page at 'https://m.momoso.com/groupbuy/today?share=true' was loaded over HTTPS, but requested an insecure image<br> 'http://7sbq7i.com1.z0.glb.clouddn.com/oss/5e2dde6eea5abd8eb084ac35461fe15b92a3a790.png'. This request has been blocked; the content must<br> be served over HTTPS.", source: https://m.momoso.com/groupbuy/today?share=true (0)<br>
主要意思是https的请求中加载http的图片，出错，结果图片没有加载出来。以后在调试bug的时候，注重一下打印，打印中会出现许多有用的信息，避免自己毫无头绪的找原因，可以很快定位。<br>
这是android API21以后才会出现的一个问题。<br>
博客：http://stackoverflow.com/a/31513802/3962533 上有解决办法。
```
    // Fix image doesn't load. See http://stackoverflow.com/a/31513802/3962533
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
      settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
```
7. Dialog消失异常<br>
在Activity中添加DialogFragment 使DialogFragment消失的方法有两种getDialog.dismiss()和dismiss()。<br>
存在这样的一种应用场景：DialogFragment跳转到Activity，在DialogFragment中startActivity然后getDialog.dismiss。在startActivity启动的过程中因异常挂掉，或者是触发了返回键，跳转的Activity销毁，再次点击跳转会出现如下的errorLog提示。<br>
原因是Dialog消失，但是Activity并没有OnResume。Activity处在onSaveInstanceState生命周期，DialogFragment再次创建就会出现bug。使用dismiss就不会出现这样的bug.<br>
```java
7091): java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
E/AndroidRuntime( 7091): at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1493)
E/AndroidRuntime( 7091): at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1511)
E/AndroidRuntime( 7091): at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:634)
E/AndroidRuntime( 7091): at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:613)
E/AndroidRuntime( 7091): at android.support.v4.app.DialogFragment.show(DialogFragment.java:156)
E/AndroidRuntime( 7091): at com.imaygou.android.settings.KefuDialogFragment.show(KefuDialogFragment.java:41)
E/AndroidRuntime( 7091): at com.imaygou.android.settings.SettingsActivity.toCustomerService(SettingsActivity.java:197)
E/AndroidRuntime( 7091): at com.imaygou.android.settings.SettingsActivity.lambda$createClickListener$339(SettingsActivity.java:143)
E/AndroidRuntime( 7091): at com.imaygou.android.settings.SettingsActivity.access$lambda$1(SettingsActivity.java)
E/AndroidRuntime( 7091): at com.imaygou.android.settings.SettingsActivity$$Lambda$2.onClick(Unknown Source)
E/AndroidRuntime( 7091): at android.view.View.performClick(View.java:5184)
E/AndroidRuntime( 7091): at android.view.View$PerformClick.run(View.java:20893)
E/AndroidRuntime( 7091): at android.os.Handler.handleCallback(Handler.java:739)
E/AndroidRuntime( 7091): at android.os.Handler.dispatchMessage(Handler.java:95)
E/AndroidRuntime( 7091): at android.os.Looper.loop(Looper.java:145)
E/AndroidRuntime( 7091): at android.app.ActivityThread.main(ActivityThread.java:5942)
E/AndroidRuntime( 7091): at java.lang.reflect.Method.invoke(Native Method)
E/AndroidRuntime( 7091): at java.lang.reflect.Method.invoke(Method.java:372)
E/AndroidRuntime( 7091): at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1400)
E/AndroidRuntime( 7091): at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1195)
D/MomosoApiService( 7091): <--- HTTP 200 https://api.momoso.com/ios/v1/page_view/update_flash_feeds?topic_id=57a82b168106e5600247f5c9 (2818ms)
```
####8. AsyncTask下载图片显示错误
AsyncTask启动任务时，当任务结束时，需要将任务的结果返回给进程，最好建立一种对应关系。<br>
下面是我在开发过程中挖的一个坑：<br>
情景：在界面上有5个缩略图，点击就可以显示出大图。<br>
点击缩略图启动一个asynctask，从网络上下载图片，下载完成后将下载结果保存，并显示在界面上。在这5张缩略图上来回切换，结果显示的大图错乱。原因是切换图片时g启动下载大图的AysncTask但是任务并没有执行完，再接着切换会把上一次下载的图片会与当前的缩略图关联，造成显示错误。
