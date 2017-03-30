## Android 接入adyen 支付
虽然说这个支付的市场很大，但是其官方文档真心不怎么样。 <br>

官方文档：https://github.com/Adyen/adyen-checkout-android<br>
官方issues: https://github.com/Adyen/adyen-checkout-android/issues<br>

照官方文档是绝对走不通的，在引入的过程中有以下几个问题：<br>

#### 1. SDK没有引入成功。


		In your build.gradle of the root directory add the following line:
		
		buildscript {
		    repositories {
		        ...
		    }
		    dependencies {
		        ...
		        classpath 'de.undercouch:gradle-download-task:2.1.0'
		    }
		}
		This plugin will allow Gradle to download the Adyen library.
		
		In your build.gradle of the app module add the following task:
		
		import de.undercouch.gradle.tasks.download.Download
		
		...
		
		task downloadAdyenLibrary(type: Download) {
		    src 'https://raw.githubusercontent.com/Adyen/AdyenCheckout-android/master/adyenpaysdk/adyenpaysdk-1.0.0.aar'
		    src 'https://raw.githubusercontent.com/Adyen/AdyenCheckout-android/master/adyenuisdk/adyenuisdk-1.0.0.aar'
		    dest('libs');
		}
		Once you run this task using ./gradlew downloadAdyenLibrary command, the adyenuisdk and adyenpaysdk .aar files will be downloaded in you libs folder.
		
		Next add the following snippet in your build.gradle of the app module:
	
		repositories {
		    flatDir {
		        dirs 'libs'
		    }
		}
这是官方提供的第一种引入的方式在app module build.gradle 配置上述引用信息，结果发现并没有引入。找不到Adyen
	
##### 解决办法
我采用直接将下载的.aar文件直接当作库引入到项目中的方法<br>
![](http://url/to/img.png)
![image](https://github.com/MerlinYu/blog/tree/master/blog_file/android/import_aar.png)

在app build.gradle 中添加： 
					
	compile project(':adyenpaysdk-1.0.0')
这才解决了引入的错误。解决这个错误之后还有其他错误在等着。<br>

#### 2. 编译错误：
错误提示：Manifest merger failed : Attribute application@name value=(....<br>
错误原因：是adyenpaysdk中定义了app_name 这个字符串在AndroidManifest merger 的时候发生错误！<br>
解决办法：增加 xmlns:tools="http://schemas.android.com/tools"工具并在application中添加：tools:replace="android:name"将有冲突的name replcace.

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          >
           <application
        		tools:replace="android:name">

#### 3. volley not found
编译通过，安装在模拟器上进行支付的时候闪退错误提示：

	java.lang.NoClassDefFoundError: Failed resolution of: Lcom/android/volley/toolbox/JsonObjectRequest;
		at adyen.com.adyenpaysdk.services.PaymentServiceImpl.fetchPublicKey(PaymentServiceImpl.java:29)
		at adyen.com.adyenpaysdk.Adyen.fetchPublicKey(Adyen.java:53)
		at co.cloudmall.android.payment.activity.AdyenActivityPresenter.adyenPay(AdyenActivityPresenter.java:61)
		at co.cloudmall.android.payment.activity.fragment.AdyenPayFragmentPresenter.adyenPay(AdyenPayFragmentPresenter.java:25)
		at co.cloudmall.android.payment.activity.fragment.AdyenPayFragment$1.onClick(AdyenPayFragment.java:66)
		at android.view.View.performClick(View.java:5207)
		at android.view.View$PerformClick.run(View.java:21177)
		at android.os.Handler.handleCallback(Handler.java:739)
		at android.os.Handler.dispatchMessage(Handler.java:95)
		at android.os.Looper.loop(Looper.java:148)
		at android.app.ActivityThread.main(ActivityThread.java:5441)
		at java.lang.reflect.Method.invoke(Native Method)
		at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:738)
		at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:628)
	Caused by: java.lang.ClassNotFoundException: Didn't find class "com.android.volley.toolbox.JsonObjectRequest" on path: DexPathList[[zip file "/data/app/co.cloudmall.android-1/base.apk"],nativeLibraryDirectories=[/data/app/co.cloudmall.android-1/lib/arm, /data/app/co.cloudmall.android-1/base.apk!/lib/armeabi-v7a, /vendor/lib, /system/lib]]
		at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
		at java.lang.ClassLoader.loadClass(ClassLoader.java:511)
		at java.lang.ClassLoader.loadClass(ClassLoader.java:469)
		... 14 more
volley not found 这是什么鬼！！
查看sdk源码，发现在sdk中使用:

	repositories {
		    flatDir {
		        dirs 'libs'
		    }
		}
去引入volley库，结果没有引入成功，所以找不到volley。。。
#### 解决办法：

	git clone https://github.com/Adyen/adyen-checkout-android.git
将SDK源码下载下来，然后使用：
![](http://url/to/img.png)
![image](https://github.com/MerlinYu/blog/tree/master/blog_file/android/import.png)
将sdk库源码加入到库中。然后在源码中修改代码：
删除下不必要的引用：

	 testCompile 'junit:junit:4.12'
    testCompile "org.mockito:mockito-core:1.+"
    testCompile "org.robolectric:robolectric:3.1-SNAPSHOT"
    testCompile "com.google.guava:guava:19.0"
    compile(name:'volley-release', ext:'aar')
    
增加对volley的引用：

	compile 'com.android.volley:volley:1.0.0'
	
也真的是够了，接入支付需要修改源代码才成功。

##### 4. NullPointerException



	java.lang.NullPointerException: Attempt to invoke virtual method 'void adyen.com.adyenpaysdk.controllers.NetworkController.addToRequestQueue(com.android.volley.Request)' on a null object reference
		at adyen.com.adyenpaysdk.services.PaymentServiceImpl.fetchPublicKey(PaymentServiceImpl.java:51)
		at adyen.com.adyenpaysdk.Adyen.fetchPublicKey(Adyen.java:53)
		at co.cloudmall.android.payment.activity.AdyenActivityPresenter.adyenPay(AdyenActivityPresenter.java:61)
		at co.cloudmall.android.payment.activity.fragment.AdyenPayFragmentPresenter.adyenPay(AdyenPayFragmentPresenter.java:25)
		at co.cloudmall.android.payment.activity.fragment.AdyenPayFragment$1.onClick(AdyenPayFragment.java:66)
		at android.view.View.performClick(View.java:5207)
		at android.view.View$PerformClick.run(View.java:21177)
		at android.os.Handler.handleCallback(Handler.java:739)
		at android.os.Handler.dispatchMessage(Handler.java:95)
		at android.os.Looper.loop(Looper.java:148)
		at android.app.ActivityThread.main(ActivityThread.java:5441)
		at java.lang.reflect.Method.invoke(Native Method)
		at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:738)
		at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:628)

查看源码：

	NetworkController.getInstance().addToRequestQueue(jsonObjectRequest);
是这句话造成了空指针异常。
NetworkController定义在adyenpayadk的Mainfest文件中：

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:name=".controllers.NetworkController" >

    </application>
    
        @Override
    public void onCreate() {
        super.onCreate();
        mInstance = this;
    }
在Mainfest.xml中文件可以看到其对 android:label="@string/app_name" app_name 做了引用，但是在我们的主工程中app moudle 的Mainfest.xml文件中使用tools:replace="android:name" 将有冲突的地方代替了所以定义在adyenpaysdk定义的 application不会被执行啊！！！
NetworkController并没有执行 onCreate函数，没有被初始化。看源码发现单例模式写的有问题啊！

    public static Adyen getInstance() {
        if(mInstance == null) {
            mInstance = new Adyen();
        }
        return mInstance;
    }
    
    public static synchronized NetworkController getInstance() {
        return mInstance;
    }

说实话这两种单例模式，在我们公司面试是通不过的，我是不会要的。问题找出来的大家自己解决吧。我的解决思路是：NetworkController没有定义成Application
##### 解决办法 

	public class NetworkController {
	
	   public static void init(Application application) {
	    if (mInstance == null) {
	      synchronized (NetworkController.class) {
	        if (mInstance == null) {
	          mInstance = new NetworkController(application);
	        }
	      }
	    }
	  }
	
	
	  public static  NetworkController getInstance() {
	    if (mInstance == null) {
	      throw new NullPointerException("please init NetworkController first!");
	    }
	    return mInstance;
	  }
	  ....
	}
上边解决的引入冲突，空指针的关于库的错误，下面还有在使用的过程中的几个坑。

1. 官方文档并没有指出APP_PUBLIC_KEY和APP_TOKEN的使用，官方文档简直是坑人的吧！

正确的使用方式是：

	   Adyen adyen = Adyen.getInstance();
	    adyen.setPublicKey(AppKey.ADYEN_PUBLIC_KEY);
	    adyen.setToken(AppKey.ADYEN_TOKEN);
	    adyen.setUseTestBackend(true);

2. adyen提供了第二次支付时可以使用之前绑定卡片的信息去支付，但是官方文档是一点都没有提怎么使用....
解决办法：第二次支付时只需要将绑定过卡片列表的：recurringDetailReference 传回给服务器，由服务器去支付。

不得不说adyen技术文档真的很坑！果然成功的东西其源代码并不怎么样！

