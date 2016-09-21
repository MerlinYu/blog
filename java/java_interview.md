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
### 10. java 线程
https://github.com/MerlinYu/blog/blob/master/java/java_thread.md
### 11. 泛型
泛型是Java SE 1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。 Java语言引入泛型的好处是安全简单。<br>
泛型的好处是在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，以提高代码的重用率。
```java
List<String> array = new ArrayList<String>();
```
### 12. 继承
- 子类在初始化时会先初始化基类
- 基类中定义的属性（变量）会先执行然后再调用构造函数。
- 如果派生类有参数构造，在构造时并没有调用super（）,会先调用基类的无参构造器。
代码测试：<br>
```java
public class Game {
  String name = Game.class.getName();
  public Game() {
    System.out.println(name);
  }
}
public class BasketballGame extends Game {
  String name = BasketballGame.class.getName();
  public BasketballGame() {
    System.out.println(name);
  }

  public BasketballGame(String country) {
    System.out.println(country);
  }
}
BasketballGame game = new BasketballGame("china");
//打印结果如下：
com.processor.test.Game
china
```
### 13. java的反射机制
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。<br>
一个好的博客地址： http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html<br>
```java
Class<?> demo = null;
Game baseGame = null;
try {
  demo = Class.forName("com.processor.test.BasketballGame");
  System.out.println("===========本类属性========");
  Field[] field = demo.getDeclaredFields();
  // field = demo.getFields(); 父类属性
  int length = field.length;
  for(int i = 0; i < length; i++) {
    // 权限修饰符
    int mo = field[i].getModifiers();
    String priv = java.lang.reflect.Modifier.toString(mo);
    // 属性类型
    Class <?> type = field[i].getType();
    System.out.println(priv + " " + type.getName() + " " + field[i].getName());
  }

  baseGame = (Game) demo.newInstance();
  System.out.println(baseGame.getGameName());
} catch (Exception e) {
  e.printStackTrace();
}
```

