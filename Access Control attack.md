# Acess control and privilege escalation

## 进攻方法
* 直接访问/admin /robots.txt
* 如果上面的方法不行，则查看后台代码，看有没有被隐藏的部分
* 仔细检查HTTP报文，看有没有隐藏的权限等之类的信息**（请求和回复报文都要看）**
* 有的网站会有防御机制，类似于`DENY: POST, /admin/deleteUser, managers`，如果没有对应权限拒绝访问
  * 可以使用`POST / HTTP/1.1 X-Original-URL: /admin/deleteUser`