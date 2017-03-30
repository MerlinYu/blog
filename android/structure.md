#android 框架
android常常提起的框架有MCP,MVP,MVVM。在工作中我感觉MCP已经被淘汰了，MVP框架很好用，MVVM是在MVP的基础上提出来的。<br>
##MVC
Module-View-Control <br>
![](http://www.jcodecraeer.com/uploads/20160414/1460565635729862.png)<br>
MVC 模式通过controler去操作Model层的数据，然后将数据返回给View。如我们所看到的View层和Modeal可以直接通信，这样会造成view层管理混乱。<br>
在Android实现中我们常将Activity作为controler和view层，这样在Activity中会既可以操作数据又可以更新view,代码少的时候还可以，代码一旦复杂会造成Activit雍肿，逻辑混乱。<br>

##MVP
Moudle-view-Presenter<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/mvp.png)<br>
* M即Model，what to show？ 也就是显示在UI上的数据，至于数据怎么来，数据库，网络等等渠道，都是属于这一层
* V即View，how to show？也就是怎么显示数据，在Android中，通常是使用xml定义这个view，将Activity当作View层。
* P即Presenter，Presenter扮演着中间联系人的作用，就好比MVC中的Controller，通常来说，Presenetr中一般会持有View和Model的引用。<br>

#####区别

MVC和MVP在实现会有所差别，MVP将代码控制逻辑单独隔离出来，而Activity只负责显示View变化的操作。实现了View层和Controler层完全解耦。

## MVVM
Model-View-ViewModel<br>


## tips
1. blank Activity空白的Activity可以利用它实现一些很巧妙的功能，比如网络请求，后台登陆...
2. app中需要对应用的异常进行监听，当有异常发生时将信息传回给服务器，方便修改bug。android中提供了一个接口类-UncaughtExceptionHandler，实现该接口功能，就可以自定义处理运行异常。
3. app当中所有的Activity应该继承一个重写的BaseAtivity这样可以在BaseActivity对Activity的周期进行管理监测，跟踪状态。eg:在BaseActivity onCreate中添加log打印this.getSimpleName，就可以在控制台监测Activity的跳转流程。
4. android ANR界面优化http://www.jianshu.com/p/1fb065c806e6
