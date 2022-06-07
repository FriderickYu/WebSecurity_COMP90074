# Server-Side Request Forgery 服务端请求伪造

## SSRF攻击原理
攻击者通常没有服务器的所有权限时，利用服务器漏洞，并以服务器的身份发送一个请求给服务器所在的内网。SSRF
攻击通常针对外部网络无法直接访问的内部系统。

因为很多`web`应用都提供了从其他服务器上获取数据的功能。而通过指定的`URL`，应用可以获取图片、下载文件等。
之所以能发起`SSRF`，是因为服务器提供了从其他服务器应用获取数据的功能，但没有对目标地址做过滤和限制。或者干脆，
诱使服务器连接到外部的任意一个系统

## SSRF的危害
A successful SSRF attack can often result in unauthorized actions or access to data within the organization, 
either in the vulnerable application itself or on other back-end systems that the application can communicate with. 
In some situations, the SSRF vulnerability might allow an attacker to perform arbitrary command execution.
An SSRF exploit that causes connections to external third-party systems might result in malicious onward attacks 
that appear to originate from the organization hosting the vulnerable application.

## SSRF攻击方式
### SSRF attacks against the server itself
攻击者通过应用发起一个`HTTP Request`给服务器，通常包含一个`URL`其中包含类似于127.0.0.1

比如一个购物应用程序，用户可以查看某件商品在某个商店中是否有库存。医用程序通过使用后端的`REST APIs`提供商品信息，
大概就是前端通过`HTTP Request`将`URL`传递给后端接口
```http request
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```
攻击者可以通过修改`stockApi`，指定一个`URL`
```http request
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

这样攻击者可以直接访问http://localhost/admin，但是一般只有经过认证的用户才能访问；所以绕过控制

每当在查询过程中，仔细观察`HTTP Request`，看是否有http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1这种URL地址，
这种地址明显是跟后端相关的。

将其修改为http://localhost，观察服务器中多了什么，看看多出来的东西包不包含链接，比如多个admin，改为http://localhost/admin

### SSRF attacks against other back-end systems
有很多时候，用户没有权限直接能访问到后端


## 如何发现SSRF漏洞

## 如何预防SSRF攻击