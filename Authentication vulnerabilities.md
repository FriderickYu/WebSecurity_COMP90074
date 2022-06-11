# 由authentication机制带来的问题
## 密码登录
通过brute-force，挨个通过intruder进行破解姓名与账号

或者可以查看不同的response，在intruder->option->Grep Extract将报错信息圈定，在进行Payload

如果发现存在IP-based brute-force protection，尝试在HTTP Request中加入`X-Forwarded-For:500`，值要不断的变化
# 由第三方验证机制带来的问题 MFA
* 如果除了要求输入正确的账号与密码，还要求填写verification code,试试能不能通过修改url和添加session直接绕开verification code
* 如果不能绕开，已知拥有一个用户的全部信息，需要绕开另外一个用户的verification code，可以这么做
  * 使用自己的账号登录，等到需填写verification code时，交一个错的
  * 用上面的HTTP Request，把username改成要绕开的用户名，然后brute force去猜他的verification code

# 如何防御