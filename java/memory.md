#java内存机制
部分转载博客：http://my.oschina.net/xiaohui249/blog/170013<br>

## JVM简介
Java虚拟机（Java Virtual Machine 简称JVM）是运行所有Java程序的抽象计算机，是Java语言的运行环境，它是Java 最具吸引力的特性之一。
Java虚拟机有自己完善的硬体架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。JVM屏蔽了与具体操作系统平台相关的信息，
使得Java程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。<br>
一个运行时的Java虚拟机实例的天职是：负责运行一个java程序。当启动一个Java程序时，一个虚拟机实例也就诞生了。
当该程序关闭退出，这个虚拟机实例也就随之消亡。如果同一台计算机上同时运行三个Java程序，将得到三个Java虚拟机实例。
每个Java程序都运行于它自己的Java虚拟机实例中。<br>
如下图所示，JVM的体系结构包含几个主要的子系统和内存区：<br>
* 垃圾回收器（Garbage Collection）<br>
负责回收堆内存（Heap）中没有被使用的对象，即这些对象已经没有被引用了。
* 类装载子系统（Classloader Sub-System）<br>
除了要定位和导入二进制class文件外，还必须负责验证被导入类的正确性，为类变量分配并初始化内存，以及帮助解析符号引用。
* 执行引擎（Execution Engine）<br>
负责执行那些包含在被装载类的方法中的指令。
* 运行时数据区（Java Memory Allocation Area）<br>
又叫虚拟机内存或者Java内存，虚拟机运行时需要从整个计算机内存划分一块内存区域存储许多东西。例如：字节码、从已装载的class文件中得到的其他信息、程序创建的对象、传递给方法的参数，返回值、局部变量等等。
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/jvm.jpg)<br>

### java内存分区
从上节知道，运行时数据区即是java内存，而且数据区要存储的东西比较多，如果不对这块内存区域进行划分管理，会显得比较杂乱无章。
程序喜欢有规律的东西，最讨厌杂乱无章的东西。<br>
根据存储数据的不同，java内存通常被划分为5个区域：程序计数器（Program Count Register）、本地方法栈（Native Stack）、方法区（Methon Area）、栈（Stack）、堆（Heap）。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/jvm.jpg)<br>
* 程序计数器(Program Count Register)<br>
又叫程序寄存器。JVM支持多个线程同时运行，当每一个新线程被创建时，它都将得到它自己的PC寄存器（程序计数器）。
如果线程正在执行的是一个Java方法（非native），那么PC寄存器的值将总是指向下一条将被执行的指令，
如果方法是 native的，程序计数器寄存器的值不会被定义。 JVM的程序计数器寄存器的宽度足够保证可以持有一个返回地址或者native的指针。<br>
* 栈(satack)<br>
又叫堆栈。JVM为每个新创建的线程都分配一个栈。也就是说,对于一个Java程序来说，它的运行就是通过对栈的操作来完成的。
栈以帧为单位保存线程的状态。JVM对栈只进行两种操作：以帧为单位的压栈和出栈操作。我们知道,某个线程正在执行的方法称为此线程的当前方法。
我们可能不知道，当前方法使用的帧称为当前帧。当线程激活一个Java方法，JVM就会在线程的 Java堆栈里新压入一个帧，这个帧自然成为了当前帧。
在此方法执行期间，这个帧将用来保存参数、局部变量、中间计算过程和其他数据。<br>
从Java的这种分配机制来看,堆栈又可以这样理解：栈(Stack)是操作系统在建立某个进程时或者线程(在支持多线程的操作系统中是线程)为这个线程建立的存储区域，该区域具有先进后出的特性。<br>
* 本地方法栈(native stack)<br>
存储本地方方法的调用状态。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/thread_data.png)<br>
* 方法区(Method Area)<br>
当虚拟机装载一个class文件时，它会从这个class文件包含的二进制数据中解析类型信息，然后把这些类型信息（包括类信息、常量、静态变量等）放到方法区中，
该内存区域被所有线程共享，如下图所示。本地方法区存在一块特殊的内存区域，叫常量池（Constant Pool），这块内存将与String类型的分析密切相关。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/native_method.png)<br>
* 堆(Heap)<br>
Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域。
在此区域的唯一目的就是存放对象实例，几乎所有的对象实例都是在这里分配内存，但是这个对象的引用却是在栈（Stack）中分配。
因此，执行String s = new String("s")时，需要从两个地方分配内存：在堆中为String对象分配内存，在栈中为引用（这个堆对象的内存地址，即指针）分配内存，如下图所示。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/stack_heap.png)<br>

### GC
Java堆是垃圾收集器管理的主要区域，因此又称为“GC 堆”（Garbage Collectioned Heap）。<br>
现在的垃圾收集器基本都是采用的分代收集算法，所以Java堆还可以细分为：新生代（Young Generation）和老年代（Old Generation），如下图所示。
分代收集算法的思想：第一种说法，用较高的频率对年轻的对象(young generation)进行扫描和回收，这种叫做minor collection，而对老对象(old generation)的检查回收频率要低很多，称为major collection。
这样就不需要每次GC都将内存中所有对象都检查一遍，以便让出更多的系统资源供应用系统使用；<br>
另一种说法，在分配对象遇到内存不足时，先对新生代进行GC（Young GC）；当新生代GC之后仍无法满足内存空间分配需求时， 才会对整个堆空间以及方法区进行GC（Full GC）。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/gc.jpg)<br>

### java内存说明
简单的来说java内存分为栈和堆两部分。JVM为每个新创建的线程都分配一个栈。Java堆是被所有线程共享的一块内存区域。在此区域的唯一目的就是存放对象实例，
几乎所有的对象实例都是在这里分配内存，但是这个对象的引用却是在栈（Stack）中分配。<br>
java 数据在内存中的存储有以下两种：<br>
- Java的基本数据类型<br>
Java的基本数据类型共有8种：int, short, long, byte, float, double, boolean, char(noti:并没有string的基本类型)。<br>
由于大小可知，生存期可知(这些字面值定义在程序块里，程序块退出后，字段值就消失了)，出于追求速度的原因，就存在于栈中。<br>
另外，栈有一个很重要的特殊性，就是存在栈中的数据可以共享。比如我们同时定义：<br>
```java
int a=3;
int b =3;
```
在栈中只会开辟一个int内存，当重新赋值时b = 5;才会再开辟另外一个int内存。<br>
- 对象<br>
在java中，创建一个对象包括对象的声明和实例化两步。<br>
```java
public class Rectangle {
     double width;
     double height;
     public Rectangle(double w,double h){
          w = width;
          h = height;
     }
}
```
(1) 声明对象<br>
　用Rectangle rect；声明一个对象rect时，将在栈内存为对象的引用变量rect分配内存空间，但Rectangle的值为空，称rect是一个空对象。空对象不能使用，因为它还没有引用任何"实体"。<br>
(2)对象实例化<br>
　当执行rect=new Rectangle(3,5)；时，首先在堆内存中为类的成员变量width,height分配内存，并将其初始化为各数据类型的默认值；接着进行显式初始化（类定义时的初始化值）；最后调用构造方法，为成员变量赋值。
　返回堆内存中对象的引用（相当于首地址）给引用变量rect,以后就可以通过rect来引用堆内存中的对象了。<br>
　概括而言：1. 在堆中分配内存，2. 将堆中的首地址赋值给栈中的引用。<br>

一个类通过使用new运算符可以创建多个不同的对象实例，这些对象实例将在堆中被分配不同的内存空间，改变其中一个对象的状态不会影响其他对象的状态。<br>
例如：
```java
Rectangle r1= new Rectangle(3,5);
Rectangle r2= new Rectangle(4,6);
```
此时，将在堆内存中分别为两个对象的成员变量width、height分配内存空间，两个对象在堆内存中占据的空间是互不相同的。如果有：<br>
```java
Rectangle r1 = new Rectangle(3,5);
Rectangle r2 = r1;
```
则在堆内存中只创建了一个对象实例，在栈内存中创建了两个对象引用，两个对象引用同时指向一个对象实例。<br>
常量池(constant pool)指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。
它包括了关于类、方法、接口等中的常量，也包括字符串常量。<br>
常量池还具备动态性，运行期间可以将新的常量放入池中，String类的intern()方法是这一特性的典型应用。<br>

## Java的内存管理实例
Java程序的多个部分(方法，变量，对象)驻留在内存中以下两个位置：即堆和栈，现在我们只关心3类事物：实例变量，局部变量和对象：<br>
实例变量和对象驻留在堆上,局部变量驻留在栈上。<br>
让我们查看一个java程序，看看他的各部分如何创建并且映射到栈和堆中：<br>
```java
public class Dog {
Collar c;
String name;
//1. main()方法位于栈上
public static void main(String[] args) {
//2. 在栈上创建引用变量d,但Dog对象尚未存在
Dog d;
//3. 创建新的Dog对象，并将其赋予d引用变量
d = new Dog();
//4. 将引用变量的一个副本传递给go()方法
d.go(d);
}
//5. 将go()方法置于栈上，并将dog参数作为局部变量
void go(Dog dog){
//6. 在堆上创建新的Collar对象，并将其赋予Dog的实例变量
c =new Collar();
}
//7.将setName()添加到栈上，并将dogName参数作为其局部变量
void setName(String dogName){
//8. name的实例对象也引用String对象
name=dogName;
}
//9. 程序执行完成后，setName()将会完成并从栈中清除，此时，局部变量dogName也会消失，尽管它所引用的String仍在堆上
}
```
