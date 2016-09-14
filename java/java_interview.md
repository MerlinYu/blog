#面试
###1. ==和equals的区别
==比较的的是两个对象的存储的位置也就是内存地址，而equals比较的是存储的内容。
###2. 面向对象编程的原则
多态，继承，封装
###3. 多态
多态是指同一个函数有多种实现，在java中有三种表现形式：1.继承重写2.java接口重写3.方法重载
###4. 静态变量加载
静态变量在运行期时加载，静态变量在第一次初始化时执行一次。
###5. 交换两个变量的值
```java
int a = 5;
int b = 7;
a = a + b;
b = a - b;
a = a - b;
```
###6. 线程同步
同步用来控制共享资源在多个线程间的访问，以保证在同一时间内只有一个线程访问该资源，在非同步线程当中一个线程正在修改一个共享变量的时候，<br>可能另一个线程也在使用或更新它的值，同步避免了脏数据的产生。<br>
方法的同步：
```java
public synchronized void method() {
}
```
代码同步：
```java
public void method() {
  synchronized (this) {
  }
}
```
###7. final finally,finalize
final 是java中的关键字表示该变量的地址不可改变
finally常与try catch一块使用，表示函数return 之后会进入到该代码块中执行。
finalize是垃圾回收器要回收对象的时候，要调用这个类的finalize方法。

###8. java的集合框架
java中集合类里面基本的接口有：<br>
* Collection
* Set
* List
* Map

###9. java HashMap 和 ArrayList
HashMap存储本质是：链表和数组结合使用。<br>
HashMap以 key -value的形式存储,它使用hashCode()和equals()方法来向集合中添加和检索元素。HashMap的初始capacity为16，负载因子（load factor）为0.8。其插入数度快，而遍历慢。<br>
ArrayList存储单纯采用数组。插入速度慢，而遍历快。
