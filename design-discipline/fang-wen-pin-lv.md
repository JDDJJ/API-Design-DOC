#访问频率
##设置的访问时间要合理
举例：客户端一般启动的时候会请求多个接口，那么当这些请求到达后，服务端可能拒绝其中一部分访问（因为在频率控制内）
##选择性限制
一般来说不需要对所有的接口都进行频率控制，仅仅针对重要的内容以及性能上有要求的接口进行频率控制。

有些功能也很复杂，如果某个接口访问量太大，会导致服务器崩溃，需要分别对每个接口每次访问设置频率（也可以统一设置每个接口访问的频率）。
##实现建议
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



