## 如何判断是否是简单请求?

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。只要同时满足以下两大条件，就属于简单请求。

l 请求方法是以下三种方法之一：HEAD,GET,POST

l HTTP的头信息不超出以下几种字段：

Accept、Accept-Language、Content-Language、Last-Event-ID、Content-Type(只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain)

凡是不同时满足上面两个条件，就属于非简单请求。
