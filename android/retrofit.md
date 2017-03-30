### Retrofit2.0
主要分为三部分来讲解Retrofit:<br>
1. Retrofit与其他库的比较； <br>
2. 原理<br>
3. 使用<br>



#### 简单使用

Retrofit 的使用分为三步：定义网络请求的接口，配置Retrofit基本参数，客户端定义Callbck调用接口请求数据。

1. 定义网络请求接口：
``` java 	
		 // TestApi
		  @GET("items/hot_keywords")
		  Call<TestKeyData> getKeyWords();
  ```


2. Retrofit 

		// RetrofitApiService
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
	        .baseUrl("http://api-test.momoso.com/9394/ios/v1/")
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
		api.getKeyWords().enqueue(callback);
		  
上面是一个使用Retrofit的示例，在其中使用Gson Convert服务器返回的数据默认解析成Gson数据。之所以贴出代码是为了之后的源码分析做准备。

#### 源码
Retrofit使用动态代理解析接口Api的请求，使用













