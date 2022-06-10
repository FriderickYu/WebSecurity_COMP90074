# CORS攻击
## SOP
[SOP是什么](https://www.jianshu.com/p/040bb5d067ec)

一个URL只有协议、域名、端口都相同的情况下，才叫同源

SOP限制不同源的URL去操作其要保护网站的数据

当浏览器发送一个HTTP Request，从一个网站到另一个，这个Request会包含cookies,session等。
这就意味着Response生产时，也会带有这些信息。没有SOP的保护，攻击者可能会盗取session和cookies

## 什么是CORS
[什么是CORS](https://www.freebuf.com/vuls/323358.html)

## CORS进攻步骤
* 观察HTTP报文，如果出现Accept: */*，则加上Origin:http://www.xxx.com
* 如果Response出现Access-Control-Allow-Origin:http://www.xxx.com，还有Access-Control-Allow-Credentials:true，则说明一定存在CORS漏洞
* 如果在Request加上Origin:http://www.xxx.com，Response里面没有出现Access-Control-Allow-Origin:http://www.xxx.com，则说明有whitelist机制防御
* 将Origin改为null
* 注入脚本

## CORS危害

## 防御CORS
### Proper configuration of cross-origin requests
If a web resource contains sensitive information, 
the origin should be properly specified in the Access-Control-Allow-Origin header.
### Only allow trusted sites
It may seem obvious but origins specified in the Access-Control-Allow-Origin header should only be sites that are trusted. 
In particular, dynamically reflecting origins from cross-origin requests without validation is readily exploitable and should be avoided.
### Avoid whitelisting null
Avoid using the header Access-Control-Allow-Origin: null. 
Cross-origin resource calls from internal documents and sandboxed requests can specify the null origin. 
CORS headers should be properly defined in respect of trusted origins for private and public servers.
### Avoid wildcards in internal networks
Avoid using wildcards in internal networks. 
Trusting network configuration alone to protect internal resources is not sufficient when internal browsers can access untrusted external domains.
### CORS is not a substitute for server-side security policies
CORS defines browser behaviors and is never a replacement for server-side protection of sensitive data
- an attacker can directly forge a request from any trusted origin. 
- Therefore, web servers should continue to apply protections over sensitive data, 
- such as authentication and session management, in addition to properly configured CORS.
