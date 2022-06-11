# 信息泄露
## 哪些东西会造成文件泄露
### 用于网络爬虫的文件
有些文章会提供类似于`/robots.txt`和`/sitemap.xml`来帮助爬虫者浏览网站。
这些文件通常会列出爬虫过程中应该跳过的一些目录。因为可能会包含敏感信息。
而这些文件不会在`HTTP`报文当中存在，需要手动添加，看看能不能导航到
### 开发者留下的注释
尤其要注意web网页当中留下的注释
### Error
通过修改`HTTP`报文里面的参数，去查看详细的报错信息
### Debug
浏览网页的同时，注意target -> site map当中的文件，发现可疑的后端文件，执行，查看执行效果
### 通过backup files盗取源代码
通过访问`/robots.txt`和`/sitemap.xml`来查看是否存在`backup`
### 因不安全的配置导致的信息泄露
* 如果通过GET无法访问`/admin`，则使用TRACE访问`/admin`
* 如果出现了`X-Custom-IP-Authorization`，看看里面的是不是自己的IP
* 点击`Proxy` -> `Option`，去`Match and Replace`，点击`add`，在`Replace`中加入`X-Custom-IP-Authorization: 127.0.0.1`
### 版本控制导致泄露
尝试在url后面加上/.git
