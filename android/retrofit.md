## Retrofit2.0源码分析
主要分为三部分来讲解Retrofit:<br>
1. Retrofit与其他库的比较； <br>
2. 使用<br>
3. 原理<br>
Retrofit是一个基于OkHttp restful api 的网络请求框架。实际上Retrofit就是充当了一个适配器（Adapter）的角色：将一个Java接口翻译成一个Http请求，然后用OkHttp去发送这个请求。<br>


### 比较
目前流行的android网络库有两个Square的Retrofi和Google的Volley，这两个库各有千秋。Retrofit的简化了网络请求流程，内部使用OkHtttp进行网络请求，Retrofit提供不同的解析器支持多种格式的解析，例如Gson。<br>
CSDN上有一篇博客说明了Retrofit和其他库的对比：
http://blog.csdn.net/carson_ho/article/details/52171976 大家可以看下。

我认为Retrofit的优点有：
1. 简洁易用，RestFul api的风格和使用注解，可以让我们只关注网络请求，而不用关心其具体流程。
2. 支持同步异步
3. 支持Gson自动解析
缺点是：高度封装，解析数据使用统一的converter来处理数据，很难处理服务器多变的数据。

没有使用过Volley,但是查找资料Volley突出的功能有：网络请求与Activity和Fragment的生命周期关联，当Acitvity和Fragment销毁时可以自动取消网络请求，扩展性好。


#### 简单使用

Retrofit 的使用分为三步：定义网络请求的接口，配置Retrofit基本参数，客户端定义Callbck调用接口去请求数据。

1. 定义网络请求接口：

		//定义TestKeyData，使用Parcelable序列化数据，使用注解@SerializedName
		//标明keyList数据是json中的keywrods
		public class TestKeyData  extends BaseResponse implements Parcelable{
		
		  @Expose
		  @SerializedName("keywords") 
		  public ArrayList<KeyWords> keyList;
		
		
		  protected TestKeyData(Parcel in) {
		    keyList = in.createTypedArrayList(KeyWords.CREATOR);
		  }
		
		  public static final Creator<TestKeyData> CREATOR = new Creator<TestKeyData>() {
		    @Override
		    public TestKeyData createFromParcel(Parcel in) {
		      return new TestKeyData(in);
		    }
		
		    @Override
		    public TestKeyData[] newArray(int size) {
		      return new TestKeyData[size];
		    }
		  };
		
		  @Override
		  public int describeContents() {
		    return 0;
		  }
		
		  @Override
		  public void writeToParcel(Parcel dest, int flags) {
		    dest.writeTypedList(keyList);
		  }
		
		  public String toString() {
		    return new StringBuffer().append("code" + code + "\n" + keyList.toString()).toString();
	  		}
		}

		 	
		 // TestApi.java
		  @GET("items/hot_keywords")
		  Call<TestKeyData> getKeyWords();


2. Retrofit 

		// RetrofitApiService.java
		  private static Retrofit mRestAdapter;
		  
		  // 初始化retrofit
		  void initRetrofit() {
		  
		  final OkHttpClient okHttpClient = new OkHttpClient.Builder()
		        .connectTimeout(30, TimeUnit.SECONDS)
		        .readTimeout(30, TimeUnit.SECONDS)
		        .cache(new Cache(application.getCacheDir(), 1024L * 1024L * 30))
		        .connectionPool(new ConnectionPool())
		        .addInterceptor(getHeaderInterceptor())
		        .addInterceptor(getLoggingInterceptor())
		        .build();
		 mRestAdapter = new Retrofit.Builder()
	        .baseUrl("http://www.github.com/v1/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
	        .client(okHttpClient)
	        .build();

		 }
		 
		 // 获取接口
		 public static <T> create(Class<T> service) {
		 	T retrofitInterface = mRestAdapter.create(service);
		 }
		 
3. 调用 
		
		TestApi api = RetrofitApiService.create(TestApi.class);
		Callback<TestKeyData> callback = new Callback<TestKeyData> {
		.....
		};
		api.getKeyWords().enqueue(callback); // 异步调用
		  
上面是一个使用Retrofit的示例，在其中使用Gson Convert服务器将返回的数据默认解析成所定义好的数据。之所以贴出代码是为了之后的源码分析做准备。

#### 源码理解

在知乎上找到一张图可以说明Retrofit的框架：<br>
![](https://pic4.zhimg.com/3ee2782a2a812423f2cc4034541e096b_r.png)<br>
其源网址是：<br>
https://www.zhihu.com/question/35189851<br>


#### 1. 通过接口APi生成动态代理

		// 生成网络请求接口
	 	T retrofitInterface = mRestAdapter.create(service);
	 	
	 	// 具体代理过程
	 	public <T> T create(final Class<T> service) {
	 	 ....
	    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
	        new InvocationHandler() {
	          private final Platform platform = Platform.get();

          @Override 
          public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  	}
  	
 从上边的代码可以知道由定义接口创建APi请求时通过Java反射生成动态代理。<br>
 在生成代理的过程中可以看到是为接口的每个method创建了一个对应的ServiceMethod,并使用这个ServiceMethod对象创建OkHttpCall,并使用ServiceMethod实例的callAdapter来调用okhttpCall并返回结果。<br>

通过上面代码可以看到调用关键的就是三步：<br>

1. 加载对应method的ServiceMethod实例.
2. 使用ServiceMethod实例和方法调用参数创建OkHttpCall.
3. 调用serviceMethod.callAdapter.adapt(okHttpCall)来产生method所定义的返回(Call<T>或者其他自定义CallAdapter支持的返回).

##### 2. 网络请求
下一步我们来分析一下如何进行网络调用。从上边的源码可以知道在动态代理的过程中生成OkHttpCall实例，来执行具体的网络请求，其关键源码如下：

	//构造OkHttpCall
	 OkHttpCall(ServiceMethod<T> serviceMethod, Object[] args) {
	    this.serviceMethod = serviceMethod;
	    this.args = args;
	  }
	  
	// 执行异步请求的过程：
	@Override 
	public void enqueue(final Callback<T> callback) {
	  ...
    okhttp3.Call call;
    Throwable failure;
    
    synchronized (this) {
      .....
       if (call == null && failure == null) {
        try {
          call = rawCall = createRawCall();
        } catch (Throwable t) {
          failure = creationFailure = t;
        }
      }
      
    call.enqueue(new okhttp3.Callback() {
      @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
          throws IOException {
        Response<T> response;
        ......
        response = parseResponse(rawResponse);
        callSuccess(response);
      }

      @Override public void onFailure(okhttp3.Call call, IOException e) {
       
        ....
        callback.onFailure(OkHttpCall.this, e);
     
      }

      private void callFailure(Throwable e) {
      	 .....
         callback.onFailure(OkHttpCall.this, e);
     
      }

      private void callSuccess(Response<T> response) {
      	......
       callback.onResponse(OkHttpCall.this, response);
   
      }
    });
    }
生成okhttp3.Call的过程需要我们一步步去逆向追究：

1. createRawCall通过serviceMethod.callFactory.newCall()生成okhttp3.Call
		
		// OkHttpCall.java 
		 private okhttp3.Call createRawCall() throws IOException {
		    Request request = serviceMethod.toRequest(args);
		    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
		    if (call == null) {
		      throw new NullPointerException("Call.Factory returned null.");
		    }
		    return call;
		  }
		  
		  
2. ServiceMethod通过Retrofit初始化callFactory
 
		ServiceMethod(Builder<T> builder) {
			this.callFactory = builder.retrofit.callFactory();
			....
		}
		
3. Retrofit通过建造者模式设置callFactory

		// 实例化Retrofit需要做此步骤
	    public Builder client(OkHttpClient client) {
	      return callFactory(checkNotNull(client, "client == null"));
	    }
	
	    public Builder callFactory(okhttp3.Call.Factory factory) {
	      this.callFactory = checkNotNull(factory, "factory == null");
	      return this;
	    }
	    
4. OkHttpClient实现接口Call.Factory返回Call
    
		@Override public Call newCall(Request request) {
		    return new RealCall(this, request);
		}
在逆向分析中可以看到Retrofit只是适配了Okhttp的功能，对其进行封装。通过源码我们可以看到在OkHttpCall中执行异步请求的过程中其实质上还是通过okhttp3.Call去执行具体的请求。

#### 3. 返回数据
在第一步当中可以看到通过ServiceMethod的Adapter对Okhttpcall进行解析：
serviceMethod.callAdapter.adapt(okHttpCall).
我们在初始化Reftrofit时添加了相应的解析器：

	.addConverterFactory(GsonConverterFactory.create())
	.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
这些解析器都实现了方法：<R> T adapt(Call<R> call);解析器的源码就没有必要在这里说了。



	
	
		     







		














