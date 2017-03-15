#API调研

##设计要点

###Restful设计原则 
一种软件架构风格，设计风格而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

- 无状态（Stateless）

  无状态协议是指协议对务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。 Http协议不像建立了socket连接的两个终端，双方是可以互相通信的，http的客户端只能通过请求服务器来获取相关内容或文件信息，具体内容可以通过我的另一篇文章了解。
  http协议这种特性有优点也有缺点，优点在于解放了服务器，每一次请求“点到为止”不会造成不必要连接占用，缺点在于每次请求会传输大量重复的内容信息。

- 客户-服务器（Client-Server）

  通信只能由客户端单方面发起，表现为请求-响应的形式

- 缓存（Cache）

  响应内容可以在通信链的某处被缓存，以尽量减少服务端和客户端之间的信息传输，以提高性能

- 统一接口（Uniform Interface）

  个REST系统需要使用一个统一的接口来完成子系统之间以及服务与用户之间的交互。这使得REST系统中的各个子系统可以独自完成演化。

- 分层系统（Layered System）

  通过限制组件的行为（即每个组件只能“看到”与其交互的紧邻层），将架构分解为若干等级的层

****

###资源认证
- shiro
- oauth2 ps：第三方服务用的
  [oauth2](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

###AccessToken

​	在整个架构设计的一个环节中，对api访问需要进行认证，token认证是一种普遍的认证手段。主要是为了让服务器唯一识别客户端。

###CSRFTOKEN
[跨站点请求伪造（CSRF）及对策	](http://www.myhack58.com/Article/html/3/7/2013/39629.htm)

![测试](http://pic002.cnblogs.com/img/hyddd/200904/2009040916453171.jpg)

> CSRF 是什么？

Cross-site request forgery 跨站请求伪造，也被称为 “one click attack” 或者 session riding，通常缩写为 CSRF 或者 XSRF，是一种对网站的恶意利用。CSRF 则通过伪装来自受信任用户的请求来利用受信任的网站。

CSRF 攻击类似 XSS 攻击，都是在页面中嵌入特殊部分引诱或强制用户操作从而得到破坏等的目的，区别就是迫使用户 `访问特定 URL / 提交表单` 还是执行 javascript 代码。

> 为什么叫 `跨站请求` 攻击？

从字面意思就可以理解：当你访问 `fuck.com` 黑客页面的时候，页面上放了一个按钮或者一个表单，URL/action 为 `http://you.com/delete-myself`，这样引导或迫使甚至伪造用户触发按钮或表单。在浏览器发出 GET 或 POST 请求的时候，它会带上 `you.com` 的 cookie，如果网站没有做 CSRF 防御措施，那么这次请求在 `you.com` 看来会是完全合法的，这样就会对 `you.com` 的数据产生破坏。

> 如何防止 CSRF ？

CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信息不存在于Cookie之中。鉴于此，系统开发者可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。



###用户登录安全
- 访问频率限制
- token生成验证规则
- https防劫持 伪造证书怎么伪造 怎么杜绝
- token时效 ps：第三方服务
- cache


> 以下两种方案是被否决的方案 仅供参考

####方案一
用户提交username + pwd后，服务端返回以下信息：



```
	{
		"access_token":"2YotnFZFEjr1zCsicMWpAA",
		"expires_in":3600,
		"refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA"
	}
```



access_token 是用来进行访问的接口的，expires_in 是他的过期时间，到达过期时间后，需要用 refresh_token 来请求服务端刷新 access_token。

这里几个重点是：refresh_token 仅能使用一次，使用一次后，将被废弃。另外这个 access_token 只在 expires_in 有效期内有效。

注意： 这里的 expires_in 仅返回秒数就好了。别返回时间戳。因为各个平台计算s的时间戳，不一致，这样子做更方便处理。
####方案二
c. client在开启成功后,会获取到服务器返回的token值, 请将header中的Authorization设置为该值, 服务端以此识别用户, 否则无权操作需要授权的接口;

d. 所有POST请求均需对参数签名 一个合法请求如下:


```

	{
        "foo": "bar",
        "a": 1,
        "ret": True,
        "ab": 1.2,
        "a1": False,
        "timestamp":1425860757
        "sign": "2455c2db93451999664f7d557f7c0716",
    }

```


其中sign具体算法为：

    对除去sign字段的key进行字母排序: ['a', 'a1', 'ab', 'foo', 'ret']

    value为bool类型时转为0或1

    拼接加密字符串: a=1&a1=0&ab=1.2&foo=bar&ret=1&token=asjfjasf

    附加密钥字段: a=1&a1=0&ab=1.2&foo=bar&ret=1&Fr46lwabMP6MNSNtJr3t8HjVNjzIVJeE&timestamp=1425860757

    对上述字符串md5得到sign: 2455c2db93451999664f7d557f7c0716
**注：经过一轮讨论 方案一属于对外做服务的token生成方案 例如微信等 不适合红小宝项目 不予考虑 而https是防止中间人劫持，篡改数据的，那么就不需要sign这个字段 不需要对数据进行加密**



###备注
以上两种方案都是对后续接口访问进行加密，而用户名+密码则是明文传输，需不需要加密呢？经过查询 有人提出用RSA非对称加密进行密码登录验证
具体方案如下：

 ![密码加密方案](http://nmgfrank.com/wp-content/uploads/2015/07/login.png)

 而HTTPS的原理就是这个：

 ![](http://upload-images.jianshu.io/upload_images/1430132-d66d71b8bc4c14b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 而经过调研 例如支付宝，淘宝这样的金融项目 有安全插件检测这一功能 从而防止密码明文丢失问题

 [密码明文](http://blog.csdn.net/attilax/article/details/8003858) https则是对数据进行加密的 密码明文则不需要加密 

 [安全插件](http://www.wosign.com/News/2016-0126-01.htm)

####总结  

​		如果利用https 则不需要对密码明文进行加密处理

###访问频率
上面我们简单实现了功能，现在app的流量上来了，有些功能也很复杂，如果某个接口访问量太大，会导致服务器崩溃，需要分别对每个接口每次访问设置频率（也可以统一设置每个接口访问的频率）。

一般我的做法是加入一个中间件。每一个接口的访问频率做好一个对应的配置文件。比如： 
* a接口 5s内可访问1次 
* b接口 10s内可访问1次（可能非常耗时，如果同时过多请求会导致服务器崩溃）

那么就把 access_token 与这些关联起来。这里需要用到Redis。当用户A进来访问了 a接口 那么设置这个token 5s内不能再次访问。



```
if ($redis->get($key)) {
        // 无法访问，还未到时间

        return ;
    }
    
    // 设置频率控制key
    $redis->setex($key, $expires, $value);
    
    // 访问接口
```




这里需要考虑几个问题：

设置的访问时间要合理。举例：客户端一般启动的时候会请求多个接口，那么当这些请求到达后，服务端可能拒绝其中一部分访问（因为在频率控制内）

一般来说不需要对所有的接口都进行频率控制，仅仅针对重要的内容以及性能上有要求的接口进行频率控制。
###API命名

- Determine what types of resources an API provides. 定义这个接口返回资源类型
- Determine the relationships between resources. 定义资源之间的关系
- Decide the resource name schemes based on types and relationships. 根据以上两条命名api
- Decide the resource schemas. 敲定资源方案
- Attach minimum set of methods to resources 针对资源的方法及要最小



###API返回数据
- 数据安全 空处理，类型安全等

####1.http返回的响应头中的错误处理
#####网络请求状态码和用户操作状态码
Status code: a three-digit number which tells the request status. To summarize, 2xx statuses means success, 4xx means client errors and 5xx means service errors. You can get a full list here or here

If you want to take just one thing from this article, make sure it’s this one: everybody freaking hates a response with HTTP 2xx and a message of error! Use the correct codes:

- 401: Unauthorized access, where the authorization process wasn’t done correctly.
- 403: Forbidden access, the client is authorized, but has no access to the resource.
- 404: The famous not found, indicating the resource is not available.
- Retrofit错误处理封装

When something fails, inform the client about what happened and how it can recover. Here’s a nice way to do it:
​	


```

	{
  		"error": {
    	"type": "Authentication Error",
   	 	"details": "Authentication could not be established with the given username and password",
   		}
	}

```


####2.响应体中的错误返回码	

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

	

```
// 正确
	data: 
	{ 
	token: 123456 
	}
	
	// 错误
	data: 123456

```


- 网络状态异常处理

  [异常处理封装](http://blog.csdn.net/sk719887916/article/details/52132106)

#### 3. 数据编码问题

- 对于服务器端返回的json数据格式，需要注意两个问题：

**一是汉字编码问题，**因为json（javascript）内部支持Unicode编码，会将汉字等转换成unicode编码保存， 所以在返回结果中，对于中文，可以直接输出中文，也可以输出中文的unicode编码，
json解析器都会很好的解析。 
比如下面两种方式都是可以的。



```
	{
	"code":"208",
	"data":"\u53c2\u6570\u4e0d\u5b8c\u6574"
	}
	
	{
	"code": "208",
	"data": "参数不完整"
	}
```



**二是字段的数据类型，**特别是数字类型的，json中尽量转成数字格式，
比如

	

```
{
	'userid':128
	}
```



不要写成

	

```
{
	'userid':'128'
	}

```



###资源 Resource
面向资源的API
- 集合包含相同类型的资源列表 例如	一个用户有联系人集合

- 资源具有一些​​状态和零个或多个子资源 每个子资源可以是简单资源或收集资源
  For example, Gmail API has a collection of users, each user has a collection of messages, a collection of threads, a collection of labels, a profile resource, and several setting resources.

- 在请求中，get是读取一个资源，post是创造一个资源 put和patch是更新一个存在的资源，delete是删除一个资源

  Request methods: the action a client wants to perform on a specific endpoint. GET is used to retrieve a resource, POST, to create one, PUT and PATCH to update an existing one, and DELETE an existing one

###版本处理
**app版本：**[语义化版本命名]
(http://semver.org/lang/zh-CN/)

**api版本：**接口不可能一成不变，在不停迭代中，总会发生变化。接口的变化一般会有几种：

	•	数据的变化，比如增加了旧版本不支持的数据类型
	•	参数的变化，比如新增了参数
	•	接口的废弃，不再使用该接口了
为了适应这些变化，必须得做接口版本的设计。实现上，一般有两种做法：

	1	每个接口有各自的版本，一般为接口添加个version的参数。
	2	整个接口系统有统一的版本，一般在URL中添加版本号，比如http://api.domain.com/v2。
大部分情况下会采用第一种方式，当某一个接口有变动时，在这个接口上叠加版本号，并兼容旧版本。App的新版本开发传参时则将传入新版本的version。

****
###避免出现空值NULL
接口的数据一般都采用JSON格式进行传输，不过，需要注意的是，JSON的值只有六种数据类型：

	•	Number：整数或浮点数
	•	String：字符串
	•	Boolean：true 或 false
	•	Array：数组包含在方括号[]中
	•	Object：对象包含在大括号{}中
	•	Null：空类型
所以，传输的数据类型不能超过这六种数据类型。以前，我们曾经试过传输Date类型，它会转为类似于"2016年1月7日 09时17分42秒 GMT+08:00"这样的字符串，这在转换时会产生问题，不同的解析库解析方式可能不同，有的可能会转乱，有的可能直接异常了。要避免出错，必须做特殊处理，自己手动去做解析。为了根除这种问题，最好的解决方案是用毫秒数表示日期。
另外，以前的项目中还出现过字符串的"true"和"false"，或者字符串的数字，甚至还出现过字符串的"null"，导致解析错误，尤其是"null"，导致App奔溃，后来查了好久才查出来是该问题导致的。这都是因为服务端对数据没处理好，导致有些数据转为了字符串。所以，在客户端，也不能完全信任服务端传回的数据都是对的，需要对所有异常情况都做相应处理。
服务器返回的数据结构，一般为：


```

	{
	code：0,
	message: "success",
	data: {
	  key1: value1,
	  key2: value2,
	   ...
	  }
	}
```


	
	•	code: 返回码，0表示成功，非0表示各种不同的错误
	•	message: 描述信息，成功时为"success"，错误时则是错误信息
	•	data: 成功时返回的数据，类型为对象或数组
	•	统一用json方面对空的处理
###图片处理
 在不同版本的app中，各种不同尺寸的手机中，同一张图片显示的尺寸可能是不一样，如果每次都需要用返回原图，然后在客户端处理，则极大浪费网络资源。而如果是后台处理好图片才返回，则又是一个挑战，怎么有效保存和裁剪多种图片尺寸呢

 例如，一开始头像只需要返回60*60的尺寸，后来在新的版本需要返回70*70, 又出了一个新版本，需要返回80*80, 每次增加一个新的尺寸，怎么在数据库上记录下来。这个问题在一开始做api的时候没考虑，后来不得不用了一个极端的方法，没增加新的图片尺寸，就在数据库中增加一个新的字段，保存并生成新的图片尺寸，结果最后数据库的头像字段有"avatar","avatar_60_60","avatar_70_70","avatar_80_80"，这种极度恶虐的设计。

 最后，针对图片，我们才用了这样的策略：

（1）客户端本地缓存图片，只有没有合适的图片，才去服务器取。

（2）当客户端需要某种尺寸的图片，由客户端告诉服务端图片的尺寸，服务端动态生成并缓存起来。

 

  例如，客户端需要图片（http://www.baidu.com/img/bdlogo.gif）的80*80的尺寸，则在图片的路径加上宽和高的参数 http://www.baidu.com/img/bdlogo.gif？w=80&h=80， 则服务器就生成80*80的尺寸并返回。

 

  采用了这样的图片处理机制，数据库中只要有一个字段保存原图就行了，其它尺寸就由客户端告诉服务端动态生成。以后无论什么尺寸的图片，数据库中都不需要记录，数据库只有原图就行了。


###在线API文档和测试
[API](http://apidocjs.com/)
