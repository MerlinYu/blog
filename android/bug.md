#android bug 总结

1. RecyclerView view holder内容显示错乱<br>
原因：RecyclerView Holder 复用，添加绑定view时如果有addview操作需要先removeallview。<br>
2. activity 打开照相机之后，状态改变<br>
原因：activity打开照相机activity被销毁，返回activity时重新Oncreate恢复activity.<br>
解决办法：1. onSaveInstanceState时保存activy数据2. 在Mainfest.xml文件设置该activity的属性：<br>
android:configChanges = "ketboardHidden|orientation|screenSize"表示屏幕横竖屏，键盘切换时该activity的状态不会改变。<br>
3. Picasso load本地图片不成功。
原因：Picasso在load图片时地址形式是URI的形式。从本地获得的图片地址如果是：storges/../...jpg必须转换。
解决办法：String str = Uri.formFile(new File(path)).toString。
4. handler sendmessage 手机黑屏
原因：在OnCreate 中使用handler.sendmessage会导致手机黑屏
5. String.formact警告
原因：String.formact()在转换时需要指定日期的形式，eg：String.formact(Local.getDefault,...)
6. null异常
原因：在许多情况下类都需要进行null判断，重复的判断，有时会导致逻辑的冗余。在服务器返回的状态中有
