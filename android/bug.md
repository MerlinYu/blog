#android bug 总结
1. RecyclerView view holder内容显示错乱<br>
原因：RecyclerView Holder 复用，添加绑定view时如果有addview操作需要先removeallview。<br>
2.activity 打开照相机之后，状态改变<br>
原因：activity打开照相机activity被销毁，返回activity时重新Oncreate恢复activity.<br>
解决办法：1. onSaveInstanceState时保存activy数据2. 在Mainfest.xml文件设置该activity的属性：<br>
android:configChanges = "ketboardHidden|orientation|screenSize"表示屏幕横竖屏，键盘切换时该activity的状态不会改变。<br>

