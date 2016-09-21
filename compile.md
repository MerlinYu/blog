#java 编译过程
相关转载博客：http://www.360doc.com/content/14/0218/23/9440338_353675002.shtml<br>
http://www.importnew.com/1796.html<br>
java有编译期和运行期，在编译期检查语法错误，生成class文件，运行期分配内存，真正的运行。<br>
常量在编译期，计算值放在常量池中，而变量在运行期分配内存在栈中，运行，计算。<br>
在开发和设计的时候，我们需要考虑编译时，运行时以及构建时这三个概念。理解这几个概念可以更好地帮助你去了解一些基本的原理。<br>
下面的代码片段中，行A和行B所标识的代码有什么区别呢？<br>
```java
public class ConstantFolding {
    static final  int number1 = 5;
    static final  int number2 = 6;
    static int number3 = 5;
    static int number4= 6;
    public static void main(String[ ] args) {
          int product1 = number1 * number2;        //line A
          int product2 = number3 * number4;        //line B
    }
}
```
在行A的代码中，product的值是在编译期计算的，行B则是在运行时计算的。
如果你使用Java反编译器（例如，jd-gui）来反编译ConstantFolding.class文件的话，那么你就会从下面的结果里得到答案。<br>
```java
public class ConstantFolding
{
  static final int number1 = 5;
  static final int number2 = 6;
  static int number3 = 5;
  static int number4 = 6;


  public static void main(String[ ] args)
  {
      int product1 = 30;
      int product2 = number3 * number4;
  }
}
```
常量折叠是一种Java编译器使用的优化技术。由于final变量的值不会改变，因此就可以对它们优化。Java反编译器和javap命令都是查看编译后的代码（例如，字节码）的利器。<br>
对发生在编译期和运行期的动作进行说明：<br>
* 方法重载--编译期<br>
方法重载也被称为编译时多态，因为编译器可以根据参数的类型来选择使用哪个方法。<br>
```java
public class {
    public static void evaluate(String param1); // method #1
    public static void evaluate(int param1);  // method #2
}
```
如果编译器要编译下面的语句的话：<br>
```java
evaluate(“My Test Argument passed to param1”);
```
它会根据传入的参数是字符串常量，生成调用#1方法的字节码<br>

* 方法覆盖--运行期<br>
方法重载被称为运行时多态，因为在编译期编译器不知道并且没法知道该去调用哪个方法。JVM会在代码运行的时候做出决定。<br>
```java
public class A {
  public int compute(int input) {        //method #3
        return 3 * input;
  }
}

public class B extends A {
  @Override
  public int compute(int input) {        //method #4
        return 4 * input;
  }
}
```
子类B中的compute(..)方法重写了父类的compute(..)方法。如果编译器遇到下面的代码：<br>
```java
public int evaluate(A reference, int arg2)  {
    int result = reference.compute(arg2);
}
```
编译器是没法知道传入的参数reference的类型是A还是B。因此，只能够在运行时，
根据赋给输入变量“reference”的对象的类型（例如，A或者B的实例）来决定调用方法#3还是方法#4.<br>
* 泛型（又称类型检验）--编译期<br>
编译器负责检查程序中类型的正确性，然后把使用了泛型的代码翻译或者重写成可以执行在当前JVM上的非泛型代码。这个技术被称为“类型擦除“。
换句话来说，编译器会擦除所有在尖括号里的类型信息，来保证和版本1.4.0或者更早版本的JRE的兼容性。<br>
```java
List<String> myList = new ArrayList<String>(10);
```
编译后成为了：
```java
List myList = new ArrayList(10);
```
* 注解（Annotation）--运行时 or 编译时。<br>
注解可以发生在编译期也可以发生在运行期，与注解的定义有关。<br>
@Override 注解<br>
```java
public class B extends A {
  @Override
    public int compute(int input){    //method #4
        return 4 * input;
    }
}
```
@Override是一个简单的编译时注解，它可以用来捕获类似于在子类中把toString()写成tostring()这样的错误。在Java 5中，用户自定义的注解可以用注解处理工具(Anotation Process Tool ——APT)在编译时进行处理。
到了Java 6，这个功能已经是编译器的一部分了。<br>
@Test注解<br>
```java
public class MyTest{
    @Test
    public void testEmptyness( ){
        org.junit.Assert.assertTrue(getList( ).isEmpty( ));
    }

    private List getList( ){
        //implemenation goes here
    }
}
```
@Test是JUnit框架用来在运行时通过反射来决定调用测试类的哪个（些）方法的注解。<br>
```java
@Test (timeout=100)
public void testTimeout( ) {
    while(true);  //infinite loop
}
```
如果运行时间超过100ms的话，上面的测试用例就会失败。<br>
```java
@Test (expected=IndexOutOfBoundsException.class)
public void testOutOfBounds( ) {
      new ArrayList<Object>( ).get(1);
}
```
如果上面的代码在运行时没有抛出IndexOutOfBoundsException或者抛出的是其他的异常的话，那么这个用例就会失败。
用户自定义的注解可以在运行时通过Java反射API里新增的AnnotatedElement和”Annotation”元素接口来处理。<br>
* 异常（Exception）---运行时异常 or 编译时异常。<br>
运行时异常（RuntimeException）也称作未检测的异常（unchecked exception），这表示这种异常不需要编译器来检测。<br>
RuntimeException是所有可以在运行时抛出的异常的父类。一个方法除要捕获异常外，如果它执行的时候可能会抛出RuntimeException的子类，
那么它就不需要用throw语句来声明抛出的异常。<br>
例如：NullPointerException，ArrayIndexOutOfBoundsException，等等。<br>
受检查异常（checked exception）都是编译器在编译时进行校验的，通过throws语句或者try{}cathch{} 语句块来处理检测异常。
编译器会分析哪些异常会在执行一个方法或者构造函数的时候抛出。
* 继承 –- 编译期，因为它是静态态。<br>
* 代理或者组合 –- 运行期，因为它更加具有动态性和灵活性。<br>


