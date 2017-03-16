###避免出现空值NULL
接口的数据一般都采用JSON格式进行传输，不过，需要注意的是，JSON的值只有六种数据类型：

- Number：整数或浮点数
- String：字符串
- Boolean：true 或 false
- Array：数组包含在方括号[]中
- Object：对象包含在大括号{}中
- Null：空类型

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


	
- code: 返回码，0表示成功，非0表示各种不同的错误
- message: 描述信息，成功时为"success"，错误时则是错误信息
- data: 成功时返回的数据，类型为对象或数组
- 统一用json方面对空的处理