# Introduction

 此文档为REST API，是提供给HoomSum的API设计文档。为Android，IOS以及Web前端提供的API，本API由前端组提供，数据源是Java后端提供。
 
# Current Version
 此设计文档为v1版文档。详细API Version设计原则，参考文档Version篇。
 
#Root EndPoint
所有 API 通过 HTTPS 与 https://api.hoomxb.com 进行通信, 所有接收和发送数据都是 JSON 格式.
所有时间格式为 ISO 8601:


```
YYYY-MM-DDTHH:MM:SSZ
```



#请求格式
很多 API 方法都有可选参数.
GET 参数应该使用 HTTP query string的方式:

```

curl -i https://api.hoomxb.com/example?state=open
```


POST, PATCH, PUT 和 DELETE 请求的参数应该使用 Content-Tpye 为 'application/json' 编码成 JSON 格式:


```
curl -i -d '{"state":"open"}' https://api.hoomxb.com/example
```
#客户端错误(HTTP 4xx)
下面是一些可能出现的客户端错误
发送错误的 JSON 格式 会收到 400 Bad Request 响应.
 

```
HTTP/1.1 400 Bad Request
 Content-Length: 32

 {"message":"请求格式错误"}
```


发送错误的 JSON 数据类型 会收到 400 Bad Request 响应.
 

```
HTTP/1.1 400 Bad Request
 Content-Length: 32

 {"message":"请求解析失败"}
```


所有的错误都含有 message 字段用来提示
HTTP 重定向(HTTP 3xx)
API 会在合适的时候使用 HTTP 重定向, 客户端应该假定所有的请求都可能响应为重定向，收到 HTTP 重定向响并不是错误的响应，客户端应该处理重定向. 重定向响应会有一个 Location header 字段包含重定向后的 URI, 客户端应该使用新的 URI 重发请求.
301 永久重定向
302 307 临时重定向
##鉴权
401
**鉴权失败/token过期**
需要登录

#响应格式


```
{
  "code": 0,
  "message": "成功",
  "data": {
    "key": "value"
  }
}

```
#必须提供 User Agent
所有的 API 请求 header 必须包括一个 User-Agent,如果没有User-Agent请求将直接被拒绝
一个合法 User-Agent 应该包括 客户端的平台，版本号等等


```
User-Agent: Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293 Safari/6531.22.7

```



