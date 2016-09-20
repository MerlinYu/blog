#Android Studio Project中添加Jni
android studro 中使用jni的实现有三步。在实现的过程中遇到一个java.lang.UnstatisfiedLinkError的问题,在网上查找相关资料并没有解决。<br>
参考：http://blog.csdn.net/niuxinlong/article/details/4176612<br>
意识到可能是自己在之前的开发过程中已经将.so库已经编译到app中，然后将app卸载，clean project重新编译，调试通过。<br>
1. 先在java中使用native，这步主要是想利用javah去生成.h的头文件。<br>
新建MyClass.java<br>
```java
public class MyJniClass {

  static {
    System.loadLibrary("jni-hello");
  }
  private native void DisplayHello();

  public void JniPrint() {
    DisplayHello();
  }
}
```
定位到该文件目录下：javac MyClass.java生成MyClass.class文件<br>

//这里需要跳出当前目录，跳到包名所在的目录然后再执行下面的命令，如果不这样执行会报一个file not found.<br>
```bash
cd ../../
javah com.example.MyClass
```
生成头文件com_example_MyClass.h<br>
使用编辑器新建MyClass.c文件<br>
代码如下：<br>
```c
#include <jni.h>
#include "com_structure_test_MyJniClass.h"
#include <android/log.h>
#include <stdio.h>

#ifndef LOG_TAG
#define LOG_TAG "ANDROID_LAB"
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)
#endif

JNIEXPORT void JNICALL Java_com_structure_test_MyJniClass_DisplayHello
        (JNIEnv *env, jobject obj)
{
    LOGE("log string from ndk.");
    return;
}
```
2. 新建jni文件夹，并将.c和.h文件转移到该文件夹下。<br>
![](https://github.com/MerlinYu/blog/tree/master/blog_file/android/flow_control/jni_1.png)
3. 配置gradle
在app/build.gradle中的defaultConfig添加代码：
ndk {
    moduleName "jni-hello"
    ldLibs "log", "z", "m"
    abiFilters "armeabi", "armeabi-v7a", "x86"
}

4. 进行编译
编译成功之后的.so库文件可以在app/build/intermediates/ndk 下查看。<br>
![](https://github.com/MerlinYu/blog/tree/master/blog_file/android/flow_control/jni_2.png)<br>
5. 运行的结果如下：<br>
![](https://github.com/MerlinYu/blog/tree/master/blog_file/android/flow_control/jni_3.png)<br>
