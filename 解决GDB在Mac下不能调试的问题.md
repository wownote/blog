"=========== Meta ============
"StrID : 28
"Title : 解决GDB在Mac下不能调试的问题
"Slug  : gdb-debug-on-mac
"Cats  : Linux
"Tags  : GDB, macOS
"Date  : 20151212T03:47:25
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
出于安性性考虑，Darwin 内核在你没有特殊权限的情况下，不允许调试其它进程。调试某个进程，意味着你对这个进程有完全的控制权限，所以为了防止被恶意利用，gdb 调试是默认禁用的。允许 gdb 控制其它进程最好的方法就是用系统信任的证书对它进行签名，本文就介绍如何生成证书并对 gdb 进行签名。

<!--more-->

在初次使用 gdb 时，可能会遇到这样的错误：

```
(gdb) run
Starting program: /usr/local/bin/fabnacci
Unable to find Mach task port for process-id 23330: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))
```

## 创建证书

按入下步骤创建代码签名的证书：

1. 打开 Keychain Access 应用程序（/Applications/Utilities/Keychain Access.app）
2. 执行菜单 **钥匙串访问** -> **证书助理** -> **创建证书**
3. 填写如下信息：
	- 名称：gdb_codesign
	- 身份类型：自签名根证书
	- 证书类型：代码签名
	- 钩选：**让我覆盖这些默认设置**
![](http://7xnua6.com1.z0.glb.clouddn.com/2015/gdb_codesigin_new.png)
4. 一路确定，直到**指定证书位置**的步骤，选择**系统**
![](http://7xnua6.com1.z0.glb.clouddn.com/2015/gdb_codesign_location.png)
5. 点击“创建”，会提示用输入系统登录密码，创建完成
6. 在**钥匙串访问程序**中，选择左侧栏的**系统**和**我的证书**，找到你刚刚创建的**gdb_codesign**证书并双击打开证书信息窗口，展开**信任**项，设置**使用此证书时：**为**始终信任**。
![](http://7xnua6.com1.z0.glb.clouddn.com/2015/gdb_codesign_trust.jpg)
7. 关闭**证书信息**窗口，系统会再次要求输入系统登录密码。

## 对 gdb 签名

执行下面的命令：

```sh
codesign -s gdb_codesign gdb
```

执行上面的命令时，系统会再次验证身份。
完成后一定要重启系统，这个很重要，否则签名不会生效。

如果出现下面的错误：

> MacBook:~ sam$ codesign -s gdb_codesign gdb
> gdb: No such file or directory

那么就指定 gdb 的全路径。


