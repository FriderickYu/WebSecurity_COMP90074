# 目录穿越攻击
## 攻击流程
* 如果网页出现图片，进行拦截，筛选image，并使用/etc/passwd进行替换，中间的../逐次增加 -> ../etc/passwd -> ../../etc/passwd
* 如果上述方法不行，使用....//etc/passwd进行代替，或者....\/etc/passwd
* 如果图片链接本身前接目录，则不要删除目录，因为服务器通常会验证前面的目录
* 有的时候服务器会验证所有文件的后缀，如果不是以xxx.png形式出现则出错
  * `../../../etc/passwd%00.png`这么解决
  
## 如何预防
The most effective way to prevent file path traversal vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. 
Many application functions that do this can be rewritten to deliver the same behavior in a safer way.

If it is considered unavoidable to pass user-supplied input to filesystem APIs, 
then two layers of defense should be used together to prevent attacks:

* The application should validate the user input before processing it. 
Ideally, the validation should compare against a whitelist of permitted values. 
If that isn't possible for the required functionality, then the validation should verify that the input contains only permitted content, 
such as purely alphanumeric characters.
* After validating the supplied input, the application should append the input to the base directory and use a platform filesystem API to canonicalize the path. 
It should verify that the canonicalized path starts with the expected base directory.
* Below is an example of some simple Java code to validate the canonical path of a file based on user input:
```php
File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // process file
}
```