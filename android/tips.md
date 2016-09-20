#android 小技巧一
##功能实现
1. 自定义statusBar 星条间隔，最好的实现办法是在切图的时候星条图片两边等间距留空。
2. bitmap.recycle()，在程序当中有中间图片产生时，在函数结束时需要使用bitmap.recycle回收资源。其回收方式为：<br>
if(bitmapObject.isRecycled()==false) //如果没有回收<br>
 bitmapObject.recycle();<br>
3. 布局控件延迟加载控件：ViewStub。
4. 布局控件合并相同的节点：merge。
5. Activity切换动画：overridePendingTransition(anim,anim);前者是进入动画，后者是退出动画。
6. view控件布局位置：view.getLocationInWindow(position);必须在onResume之后才可以得正确的值。
7. 查看app运行最大内存：max = (int) (Runtime.getRuntime().maxMemory() / 1024)
8. 图像占用内存算法: memory = width * height * config;  eg: config = ARGB_888(4)

