"=========== Meta ============
"StrID : 23
"Title : 关于SSH免密登录不生效的解决办法
"Slug  : ssh-login-without-password
"Cats  : Linux
"Tags  : Linux, SSH
"Date  : 20170212T05:55:17
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
折腾了好久，记录下来备忘。

如果公钥已经添加到 `authorized_keys` 文件了，但是 SSH 登录仍然要求输入密码的话，应该就是跟访问权限有关了。

- `.ssh` 目录的所有者和所有组都必需要当前用户及其所在组
- `.ssh` 目录的访问权限必需是 **700**
- `authorized_keys` 文件的访问权限必须是 **600**

