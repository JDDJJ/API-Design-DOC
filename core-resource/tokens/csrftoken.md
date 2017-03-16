#CSRFTOKEN
##CSRF介绍
[跨站点请求伪造（CSRF）及对策	](http://www.myhack58.com/Article/html/3/7/2013/39629.htm)

![测试](http://pic002.cnblogs.com/img/hyddd/200904/2009040916453171.jpg)

> CSRF 是什么？

Cross-site request forgery 跨站请求伪造，也被称为 “one click attack” 或者 session riding，通常缩写为 CSRF 或者 XSRF，是一种对网站的恶意利用。CSRF 则通过伪装来自受信任用户的请求来利用受信任的网站。

CSRF 攻击类似 XSS 攻击，都是在页面中嵌入特殊部分引诱或强制用户操作从而得到破坏等的目的，区别就是迫使用户 `访问特定 URL / 提交表单` 还是执行 javascript 代码。

> 为什么叫 `跨站请求` 攻击？

从字面意思就可以理解：当你访问 `fuck.com` 黑客页面的时候，页面上放了一个按钮或者一个表单，URL/action 为 `http://you.com/delete-myself`，这样引导或迫使甚至伪造用户触发按钮或表单。在浏览器发出 GET 或 POST 请求的时候，它会带上 `you.com` 的 cookie，如果网站没有做 CSRF 防御措施，那么这次请求在 `you.com` 看来会是完全合法的，这样就会对 `you.com` 的数据产生破坏。

> 如何防止 CSRF ？

CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信息不存在于Cookie之中。鉴于此，系统开发者可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。
##APP是否需要CSRFTOKEN
