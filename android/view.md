# android view 源码理解
相关博客：http://blog.csdn.net/yanbober/article/details/46128379/<br>
###Activity View层级
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/activity_view_level.png)<br>
(图片转载：http://blog.csdn.net/yanbober/article/details/46128379/)<br>
如上所示是Activity中view 的层级结构，Activity的布局文件xml显示的只是其中content的布局，此外还有titlebar和statusbar.Activity的最底层的布局是：<br>
DecorView这是一个FrameLayout。<br>
View在自定义的时候需要关注其绘制过程，view的绘制主是是三个步骤：measure,layout,draw。在源码中View特别注释了：onSizeChanged,onKeyDown,onTrackEvent,<br>
onFocusChanged,onWindowChanged,onAttachWindow,onDeatchWindow,onWindowVisibilityChanged,这些函数。这些函数与view的变化有关。<br>
下面我们将从view的绘制流程Event事件的分发讲述源码。
###view 绘制流程
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/view_draw.png)<br>
上图说明了view的绘制流程，其中performTraversals是在ViewGroup中触发的。<br>
1. measure()<br>
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/measure.png)<br>
测量并计算出view的大小，我们只需要重写其中的onMeasure方法，在其中计算出高宽，并调用setMeasureDimension使height和width起作用。<br>
2. layout()<br>
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/layout.png)<br>
在布局时会根据margin和padding来布局，由此可能会影响witdh和height.在view中有两对值onMeasureHeight，onMeasureWidth;height,width这两对值分别对应的是：<br>
measure之后的宽高和布局之后的宽高。自定义时只需要重写onlayout即可。<br>
3. draw()<br>
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/draw.png)<br>
自定义时只需要重写onDraw即可。
###view的软硬件加速
在看view源码时发现view在绘制过程中与软硬件绘制的级别有关。view中有三种绘制级别：LAYER_TYPE_SOFTWARE，LAYER_TYPE_HARDWARE，LAYER_TYPE_NONE；<br>
http://blog.csdn.net/linghu_java/article/details/41987475<br> 这篇博客详细介绍了三种方式的不同。view绘制时的layerType的值从上层得到，也就是说如果设置Activity的绘制级别为SOFTWARE则该activity中的view都以此方式绘制。
### view attach wondow
1. attach window<br>
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/view_attach_window.png)<br>
绑定view的操作是在dispatchAttachedToWindow中进行的，依次会进行的操作如下，中间省略了一些判断和步骤。<br>
在onAttachedWindow时可以看到如果isFocused==true时则从InputMethodManager中获取键盘输入事件。<br>
我们可以onAttachedWindow中做一些内部回调的绑定。<br>
2. deattach window<br>
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/view_deatch_window.png)<br>
从上面可以看出在detachedWindow时需要移除回调函数并清空drawingCache。<br>
在view 中有一个方法getWindowAttachCount 返回值是该view与一个window绑定的次数，可以用它来说明recycler view中viewholder的循环使用次数.<br>

### view motion event运动事件
![](https://github.com/merlinyu/blog/raw/master/blog_file/android/ui/view_draw/motion_event_processor.png)<br>
从上图可以看出在处理屏幕的Motion事件时首先会判断是不是isOnTouchEvent然后将事件根据结果分发处理。<br>
如果touchListener为空，则默认使用本类的OnTouchEvent来处理事件，然后再处理click事件和long click事件。<br>
在OnTouch中进行到最后一个流程要对MotionEvent事件进行处理在处理的过程同时会处理的是OnClick和OnLongClick事件。其中OnLongClick事件处理的结果会影响到OnClick事件。其关键流程如下：<br>
* onTouch switch<br> 
ACTION_UP时需要判断longLongClick事件的结果mHasPerformedLongPress。如果为false则触发preformClick如果想让View可以同时处理longclick和click事件LongClickListener的返回结果为false。ACTION_DOWN时CheckForLongClick.<br>
```java
switch(action) {
case MotionEvent.ACTION_UP:
     if(!mHasPerformedLongPress ) {
          …..
           performClick();
     }
     break;
case MotionEvent.ACTION_DOWN:
     …...
     checkForLongClick();
     break;
}
```

* check LongClick过程延时发送一个long click事件<br>
```java
checkForLongClick() {
             ….
     postDelayed(mPendingCheckForLongPress,
        ViewConfiguration.getLongPressTimeout() - delayOffset);
}
```

* CheckForLongPress中具体执行long click事件并将结果赋值给mHasPerformLongPress.<br>
```java
private final class CheckForLongPress implements Runnable {
    private int mOriginalWindowAttachCount;

    @Override
    public void run() {
        if (isPressed() && (mParent != null)
                && mOriginalWindowAttachCount == mWindowAttachCount) {
            if (performLongClick()) {
                mHasPerformedLongPress = true;
            }
        }
    }

    public void rememberWindowAttachCount() {
        mOriginalWindowAttachCount = mWindowAttachCount;
    }
}
```
如果想让OnLongClick不影响OnClick事件的执行，OnLongClick事件返回的结果必须为false.<br>
以上就是OnTouchEvent事件的处理流程。



