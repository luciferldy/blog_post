+++
date = "2015-06-16T19:00:52+08:00"
menu = "main"
tags = ["android"]
title = "HttpClient 和 HttpURLConnection 的区别"

+++

今年参加BAT春招的时候公司都有问到了 HttpClient 和 HttpURLConnection 的区别，然而我平时在用到的时候并没有太大的区别，最近读了刘志勇童鞋的一篇博客[HttpClient和HttpURLConnection的区别](http://lzyblog.com/2014/09/12/HttpClient%E5%92%8CHttpURLConnection%E7%9A%84%E4%BD%BF%E7%94%A8%E5%92%8C%E5%8C%BA%E5%88%AB-%E4%B8%8A/)，从里面摘取了一些片段对HttpClient和HttpURLConnection的分析。

##### HttpClient

HttpClient: Apache 公司提供的库。 HttpClient 最重要的功能是执行 Http 方法，一个 Http 方法的执行包含一个或者多个 Http 请求 Http 服务器响应交流，通常由 HttpClient 内部来处理。用户实例化一个请求对象，HttpClient 发送请求到目标服务器，希望服务器返回一个相应的响应对象，或者抛出一个异常。

一个简单的请求执行过程的示例

	HttpClient httpClient = new DefaultHttpClient();
	HttpGet httpGet = new HttpGet("http://localhost/");
	HttpResponse response = httpClient.execute(httpClient);
	HttpEntity entity = response.getEntity();
	if (entity != null){
		InputStream instream = entity.getContent();
		int l;
		byte[] tmp = new byte[2048];
		while((l=instream.read(tmp)) != -1)
	}

##### HttpURLConnection

HttpURLConnection: Sun 公司提供的库，一个简单的示例：

	URL url = new URL("http://localhost:8080/TestHttpConnection/index.php");
	URLConnection urlConnection = url.openConnection();
	HttpURLConnection httpURLConnection = (HttpURLConnection)urlConnection;
	// 对象参数的设置
	// 设置向httpURLConnection输出，默认情况是false
	httpURLConnection.setDoOutput(true);
	// 设置从httpURLConnection读入，默认情况是true
	httpURLConnection.setDoInput(true);
	// 不使用缓存
	httpURLConnection.setUserCaches(false);
	// 设定传送的内容类型是可序列化的Java对象
	httpURLConnection.setRequestProperty("Content-type", "application/x-java-serilized-object");
	// 设定请求的方式是post
	httpURLConnection.setRequestMethod("POST");
	httpURLConnection.connect();

	// 此处的getOutputStream会隐含的进行connect，所以不调用上一行的也可以
	OutputStream outputStream = httpURLConnection.getOutputStream();
	// 通过构建输出流对象
	ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
	// 写数据，这些数据会暂时存在内存缓存区里
	objectOutputStream.writeObject(new String("Hello World!"));
	// 清空输出流
	objectOutputStream.flush();
	// 关闭输出流，调用下面的getInputStream()函数时才把准备好的Http请求正式发送到服务器
	objectOutputStream.close();
	
	// 将post请求发送出去了
	InputStream inputStream = httpURLConnection.getInputStream();
	

	

##### HttpURLConnection 的使用注意事项

1. HttpURLConnection 的 `connect()` 函数实际上只是建立了一个与服务器的tcp连接，并没有实际发送http请求。无论是 `post` 还是 `get` 请求，`http` 请求实际上直到 HttpURLConnection 的 `getInputStream()` 这个函数里面才正式发送出去。所以对 `outputStream` 的写操作，必须在 `inputStream` 的读操作之前。
2. 在用 `post` 方式发送 `URL` 请求时，`URL` 参数的设定顺序很重要，对 `connection` 对象的一切配置，都必须要在 `connect()` 函数执行之前完成。