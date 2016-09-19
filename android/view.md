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



