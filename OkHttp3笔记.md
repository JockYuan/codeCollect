# OkHttp3 笔记

### 导入Jar包
		okHttp-3.3.0.jar
		okio-1.8.0.jar


### gradle方式:

		complile 'com.squareup.okHttp3:okhttp:3.3.0'

### Get请求:
```java
	String url = "https://www.baidu.com/";
	OkHttpClient okHttpClient = new OkHttpClient();
	Request request = new Request.Builder().url(url).build();

	Call call = okHttpClient.newCall(request);

	try{
		Response response = call.execute();
		System.out.println(response.body().string));
		}


	//header请求添加参数可以添加到Request 对象中
	Request request = new Request.Builder()
		.url(url)
		.header("key", "value")
		.header("key", "value")
		...
		.build();

	//返回结果 response 的body 可以有多种输出方法, string()只是其中之一
	response.body().bytes()
	response.code() 返回结果状态码
```
### Post请求:
```java
	String url = "https://www.baidu.com/";
	OkHttpClient okHttpClient = new OkHttpClient();

	RequestBody body = new FormBody.Builder()
		.add("key", "value")
		.add("key", "value")
		...
		.build();

	Request request = new Request.Builder()
		.url(url)
		.post(body)
		.build();

	Call call = okHttpClient.newCall(request);
	try {
		Response response = call.execute();
		System.out.println(response.body().string());
	} catch (IOException e) {
		e.printStackTrace();
	}
```
Post请求需要提交一个表单RequestBody, 请求方式和Get方法一样


RequestBody的数据格式要指定Content-Type, 常见的有三种:
* application/x-www-form-urlencode
* multipart/form-data
* application/json

FormBody 继承了RequestBody, 并指定数据类型为application/x-www-form-urlencoded

表单是个Json:
```java
		MediaType JSON = MediaType.parse("application/json; charset=utf-8");
		RequestBody body = RequestBody.create(JSON, "JSON 字串")

	  // 数据包含文件:
		RequestBody requestBody = new MultipartBody.Builder()
			.setType(MultipartBody.FORM)
			.addFormDataPart("file", file.getName(), RequestBody.create(
				MediaType.parse("image/png",file)))
			.build();

	  // MultipartBody继承了RequestBody, 它适用于五种Content-Type:

		public static final MediaType MIXED = MediaType.parse("multipart/mixed");
		public static final MediaType ALTERNATIVE = MediaType.parse("multipart/alternative");
		public static final MediaType DIGEST = MediaType.parse("multipart/digest");
		public static final MediaType PARALLEL = MediaType.parse("multipart/parallel");
		public static final MediaType FORM = MediaType.parse("multipart/form-data");
```

### 同步与异步

之前的call.execute()就是在执行http请求, 是个同步操作, 是在主线程中运行

OkHttp异步使用enqueue(CallBack callback) 加入调用队列等待调用返回
```java
		String url = "https://www.baidu.com/";
		OkHttpClient okHttpClient = new OkHttpClient();
		Request request = new Request.Builder()
				.url(url)
				.build();
		Call call = okHttpClient.newCall(request);
		call.enqueue(new Callback() {
			@Override
			public void onFailure(Call call, IOException e) {
				e.printStackTrace();
			}

			@Override
			public void onResponse(Call call, Response response) throws IOException {
				System.out.println("我是异步线程,线程Id为:" + Thread.currentThread().getId());
			}
		});

```
onFailure()和onResponse()分别是在请求失败和成功时会调用, 是在异步线程里执行的

### 自动管理Cookie
Request中可以设置Cookie
```java
		Request request = new Request.Builder()
			.url(url)
			.header("Cookie", "xxx")
			.build()
```
从返回的response里得到新的Cookie,
OkHttp可以不用自己去管理Cookie, 自动携带, 保存和更新Cookie
在创建OkHttpClient时, 设置管理Cookie的CookieJar
```java
private final HashMap<String, List<Cookie>> cookieStore = new HashMap<>();
OkHttpClient okHttpClient = new OkHttpClient.Builder()
  .cookieJar(new CookieJar() {
      @Override
      public void saveFromResponse(HttpUrl httpUrl, List<Cookie> list) {
          cookieStore.put(httpUrl.host(), list);
      }

      @Override
      public List<Cookie> loadForRequest(HttpUrl httpUrl) {
          List<Cookie> cookies = cookieStore.get(httpUrl.host());
          return cookies != null ? cookies : new ArrayList<Cookie>();
      }
  })
  .build();

```
### 响应缓存

缓存目录是私有的, 只允许一个client 访问
```java
		Cache cache = new Cache(cacheDirectory, cacheSize);
		OkHttpClient.Builder builder = new OkHttpClient.Builder();
		builder.cache(cache);
		OkHttpClient client = builder.build();
```

### 超时

OkHttp支持连接, 读取和写入超时
```java
		OkHttpClient.Builder builder = new OkHttpClient.Builder();
		OkHttpClient client = builder.build();

		client.newBuilder().connectTimeout(10, TimeUnit.SECONDS);
		client.newBuilder().readTimeout(10, TimeUnit.SECONDS);
		client.newBuilder().writeTimeout(10, TimeUnit.SECONDS);
```


### 源码分析

重要的类

Request 封装一个HTTP请求 , 包括 url, method, header, body, tag(Object),

Call  一个调用请求, 这个是接口, 代表一个请求准备执行, 这个对象代表一个请求/答复对, 不能执行两次,
	 可以被取消,
	 主要方法: execute, enqueue(CallBack responseCallback)

  RealCall 一个Call的具体实现, 传入参数OkHttpClient Request 两个对象.

	当请求网络的时候, 需要用OkHttpClient.newCall(request)进行execute和enqueue 操作,
```java
	@Override public Call newCall(Request request) {
		return RealCall.newRealCall(this, request, false /* for web socket */);
	}

	// 返回的是RealCall对象
	RealCall 的 enqueue()

		@Override public void enqueue(Callback responseCallback) {
		synchronized (this) {
			if (executed) throw new IllegalStateException("Already Executed");
				executed = true;
			}

			client.dispatcher().enqueue(new AsyncCall(responseCallback));
		}

	// 使用了OkHttpClient中Dispatcher对象的enqueue()

	// Dispatcher的本质是异步请求的管理器, 控制最大请求的并发数和单个主机的最大并发数,
	// 并持有一个线程池来执行异步请求. 对于同步的请求只用做统计.
  //
	// Dipatcher主要用于控制并发请求的变量;

		/** 最大并发请求数*/
		private int maxRequests = 64;
		/** 每个主机最大请求数*/
		private int maxRequestsPerHost = 5;
		/** 消费者线程池 */
		private ExecutorService executorService;
		/** 将要运行的异步请求队列 */
		private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
		/**正在运行的异步请求队列 */
		private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
		/** 正在运行的同步请求队列 */
		private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```
