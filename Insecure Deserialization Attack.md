# 不安全反序列化攻击
[不安全的反序列化简介](https://www.jianshu.com/p/fa912ce0426f)
## 序列化和反序列化
序列化：将内存中的对象转换成字符串的过程

反序列化：将字符串转换成对象

不同语言序列化的也不一样，有些是转换成字节，有些是转换成字符串

比如

```java
yutianqi = Person(24, 'male')
age = yutianqi.age
gender = yutianqi.gender
```

## 为什么会出现不安全的反序列化
如果反序列化的这个过程被攻击者控制，修改过的数据会被存到内存中的对象

比如游戏当中，`{'a': 100, 'b': 50}`

经过修改，`{'a': 100, 'b': 900}`

过程：攻击者在网站上拿到数据 -> 对数据进行修改 -> 数据被存到内存里的类中 -> 造成破坏

## 不安全的反序列化造成的后果
The impact of insecure deserialization can be very severe because it provides an entry point to a massively increased attack surface. 
It allows an attacker to reuse existing application code in harmful ways, resulting in numerous other vulnerabilities, often remote code execution.

Even in cases where remote code execution is not possible, insecure deserialization can lead to privilege escalation, 
arbitrary file access, and denial-of-service attacks.

## 如何防止这个漏洞
Generally speaking, deserialization of user input should be avoided unless absolutely necessary. The high severity of exploits that it potentially enables, 
and the difficulty in protecting against them, outweigh the benefits in many cases.

implement a digital signature to check the integrity of the data. 
However, remember that any checks must take place before beginning the deserialization process. 
Otherwise, they are of little use.

you should avoid using generic deserialization features altogether. 
Serialized data from these methods contains all attributes of the original object, 
including private fields that potentially contain sensitive information. 
Instead, you could create your own class-specific serialization methods 
so that you can at least control which fields are exposed.

## 如何发现此漏洞
仔细观察`HTTP`报文当中是否存在serialized data，如果有，那很可能存在该漏洞

PHP实例
```php
$user->name = "carlos";
$user->isLoggedIn = true;
```

序列化之后
```php
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

在`PHP`中，序列化使用`serialize()`和`unserialize()`

Java实例，序列化使用二进制字节码，所以很难读，注意`java.io.Serializable`



## 不安全的反序列化进攻流程
* 观察cookie，使用`base64`进行解码观察是否为`json`，如果是，修改然后发送
* 如果解码之后发现有session这种没法改，但是很重要的字段；修改data_type:1，具体数据为0
