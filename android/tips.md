#android 小技巧一
##UI布局
1. 自定义statusBar 星条间隔，最好的实现办法是在切图的时候星条图片两边等间距留空。
2. bitmap.recycle()，在程序当中有中间图片产生时，在函数结束时需要使用bitmap.recycle回收资源。其回收方式为：<br>
if(bitmapObject.isRecycled()==false) //如果没有回收<br>
 bitmapObject.recycle();<br>
3. 布局控件延迟加载控件：ViewStub。
4. 布局控件合并相同的节点：merge。
5. Activity切换动画：overridePendingTransition(anim,anim);前者是进入动画，后者是退出动画。
6. view控件布局位置：view.getLocationInWindow(position);必须在onResume之后才可以得正确的值。
7. LinerLayout devider的运用。http://gold.xitu.io/entry/55272f6ae4b0da2c5deb7f26
8. ListView逐行刷新是通过ViewHolder实现的在ListView的Adapter getView当中new viewholder,view.setTag(viewholder)然后通过position得到相应的ViewHolder来刷新UI.

## 功能
1. 查看app运行最大内存：max = (int) (Runtime.getRuntime().maxMemory() / 1024)
2. 图像占用内存算法: memory = width * height * config;  eg: config = ARGB_888(4)
3. Handler静态实现可以避免内存泄漏
```java
static class HornHandler extends Handler {
BroadCastViewHolder viewHolder;
public HornHandler(BroadCastViewHolder holder) {
viewHolder = holder;
}

@Override
public void handleMessage(Message msg) {
switch (msg.what) {
case MESSAGE_SHOW:
viewHolder.setRecyclerAnim(msg.arg1, msg.arg2);
break;
default:break;
}
super.handleMessage(msg);
}
}
```

## 逻辑
1. if循环语句优化，确认逻辑的主流程，如果条件不满足的话就直接return跳出if逻辑，或者将共有的if判断逻辑封装。

