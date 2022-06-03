# CSRF Cross-site request forgery 跨站请求伪造
## CSRF的原理
利用了站点对于合法用户的信任，以用户的名义进行非法的操作

比如

1. `用户C`打开一个浏览器，通过其访问受信任的`网站A`，输入正确的账号和密码登录`网站A`
2. 在用户信息通过验证之后，`网站A`产生`cookie`并返回给用户的浏览器，此时`用户C`登录成功
3. `用户C`在未退出`网站A`之前，在同一个浏览器中，访问`网站B`
4. 这个`网站B`为攻击者搭建的恶意网站；`网站B`在接收到用户请求后，返回一些恶意代码，并发起一个请求要求访问`网站A`
5. 浏览器接到恶意代码后，其中的`cookie`被盗取，这时`网站B`会携带`用户C`的`cookie`，向`网站A`发起一个请求
6. `网站A`并不知道新的请求是攻击者发出，还认为是`用户C`发出的，会根据`用户C`的`cookie`正常执行

一般情况下，会以受害者的名义，发送邮件、更改密码、购物和转账等

综上可总结，发起CSRF的必要条件
* 受害者必须处于登录状态
* 受害者必须要访问攻击者提供的链接/网站

Example

这是一个允许用户修改邮箱的应用，当用户发起修改邮箱的请求时，`HTTP Request`如下
```http request
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```
该实例满足了以下条件
* 修改邮箱这种操作攻击者会比较感兴趣，毕竟涉及到隐私
* `HTTP Request`中包含了`cookie`，并且没有其他的例如`token`机制来代替
* 攻击者可以很轻易的修改其中的参数

攻击者搭建一个网页
```html
<html>
<body>
<form action="https://vulnerable-website.com/email/change" method="POST">
    <input type="hidden" name="email" value="pwned@evil-user.net" />
</form>
<script>
    document.forms[0].submit();
</script>
</body>
</html>
```
当受害者访问该恶意网站时
* 恶意网站会向https://vulnerable-website.com/email/change"发送一个`HTTP Request`
* 当用户登录网站时，浏览器会把`cookie`自动的包含进来

## 如何发起CSRF攻击 && CSRF攻击的流程
### 搭建CSRF攻击
1. 使用`Burp Suite`把查看指定网页的请求报文，观察以下参数
   1. 是否是关于敏感信息的修改
   2. 是否包含cookie
   3. 各项参数是否可预期
2. 右键转发到`Repeater`，并选择`Engagement tools` -> `Generate CSRF PoC`
3. 在右上角的`Options`中选择`include auto-submit script`
4. 点击`Regenerate`，并点击`Test in browser`
### How to deliver a CSRF exploit
在这方面`CSRF`与`XSS`是一样的。一般来说攻击者将恶意的`HTML`放到恶意网站上，诱使受害者访问该网站；链接也可以
如果是使用`GET`方法，就可以用`URL`的方式，比如下面，直接使用链接把参数换掉
```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```