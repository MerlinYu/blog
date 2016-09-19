#android 框架
android常常提起的框架有MCP,MVP,MVVM。在工作中我感觉MCP已经被淘汰了，MVP框架很好用，MVVM是在MVP的基础上提出来的，然而感觉并无什么卵用。<br>
##MVC
Module-View-Control
##MVP
Moudle-view-Presenter<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/android/flow_control/mvp.png)<br>
* M即Model，what to show？ 也就是显示在UI上的数据，至于数据怎么来，数据库，网络等等渠道，都是属于这一层
* V即View，how to show？也就是怎么显示数据，在Android中，通常是使用xml定义这个view，一般View中会持有Presenter的引用。
* P即Presenter，Presenter扮演着中间联系人的作用，就好比MVC中的Controller，通常来说，Presenetr中一般会持有View和Model的引用。
##MVVM
Model-View-ViewModel
##mark
待完善
