#android第三库整理
##控件
###1. DecimalFormat
数字显示分隔，例如需要在TextView显示这样的数字4，333，此时可以用DecimalFormat来帮助实现。
##UI库
###1. CircleImageView 
这是一个圆角且带白边的ImageView使用效果如下：
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/ui/circleImage.png)
<br>
使用的方法是：
```xml
<de.hdodenhof.circleimageview.CircleImageView
android:layout_width="160dp"
android:layout_height="160dp"
android:layout_centerInParent="true"
android:src="@drawable/demo"
app:border_width="2dp"app:border_color="@color/dark" />
```
博客地址：http://www.tuicool.com/articles/mQNFJ3<br>
库：de.hdodenhof:circleimageview:1.2.1
###2. FlowLayout
这是一个流线型布局的UI,使用效果如下：<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/ui/flowlayout.png)<br>
库："org.apmem.tools:layouts:1.9@aar"
###3. GPUImage
可以实现各种各样的滤镜处理效果<br>
库："jp.co.cyberagent.android.gpuimage:gpuimage-library:1.2.3”<br>

##框架
###1. RxJava
###2. EventBus
###3. Picasso
###4. ButterKnife
###5. Retrofit

##插件



