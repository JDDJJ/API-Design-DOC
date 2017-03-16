#API返回数据
- 数据安全 空处理，类型安全等

##响应头中的错误处理
###网络请求状态码和用户操作状态码
Status code: a three-digit number which tells the request status. To summarize, 2xx statuses means success, 4xx means client errors and 5xx means service errors. You can get a full list here or here

If you want to take just one thing from this article, make sure it’s this one: everybody freaking hates a response with HTTP 2xx and a message of error! Use the correct codes:

- 401: Unauthorized access, where the authorization process wasn’t done correctly.
- 403: Forbidden access, the client is authorized, but has no access to the resource.
- 404: The famous not found, indicating the resource is not available.
- Retrofit错误处理封装

When something fails, inform the client about what happened and how it can recover. Here’s a nice way to do it:
​	

	{
  		"error": {
    	"type": "Authentication Error",
   	 	"details": "Authentication could not be established with the given username and password",
   		}
	}

##响应体中的错误返回码	

不同错误需要定义不同的返回码，属于客户端的错误和服务端的错误也要区分，比如1XX表示客户端的错误，2XX表示服务端的错误。这里举几个例子：

	•	0：成功
	•	100：请求错误
	•	101：缺少appKey
	•	102：缺少签名
	•	103：缺少参数
	•	200：服务器出错
	•	201：服务不可用
	•	202：服务器正在重启
错误信息一般有两种用途：一是客户端开发人员调试时看具体是什么错误；二是作为App错误提示直接展示给用户看。主要还是作为App错误提示，直接展示给用户看的。所以，大部分都是简短的提示信息。
data字段只在请求成功时才会有数据返回的。数据类型限定为对象或数组，当请求需要的数据为单个对象时则传回对象，当请求需要的数据是列表时，则为某个对象的数组。这里需要注意的就是，不要将data传入字符串或数字，即使请求需要的数据只有一个，比如token，那返回的data应该为：

	// 正确
	data: 
	{ 
	token: 123456 
	}
	
	// 错误
	data: 123456

## 网络状态异常处理
- Android 对异常处理进行组件化
  [异常处理封装](http://blog.csdn.net/sk719887916/article/details/52132106)
