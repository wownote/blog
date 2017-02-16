"=========== Meta ============
"StrID : 36
"Title : WordPress主题的本地化（多语言翻译）
"Slug  : wordpress-localization
"Cats  : 折腾记
"Tags  : Localization, WordPress
"Date  : 20170215T16:13:06
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
阿米巴的主题基本已成型，抽空了解了一下WordPress主题的本地化，略有复杂。业界基本是在使用一款名为 [Poedit](https://poedit.net) 的软件进行翻译，这是款收费软件，且价格不菲，专业版128块大洋。有个免费版本，只能对已有的模板文件（POT）进行翻译，且新增文案后，原有的翻译不能记忆，要重头开始翻译。忽然之间就有了自己开发一款翻译软件的想法，在这之前，先把翻译的基本概念和过程记录下来备忘。

<!--more-->

## 本地化框架

WordPress使用了 [GNU gettext](https://www.gnu.org/software/gettext/) 做为本地化的框架，**GNU gettext**是一款为其它软件包提供多语言翻译支持的框架，主要包括：

- 支持多语言的编码规范；
- 翻译文件格式规范；
- 读取翻译文案的运行时库；

大致的翻译过程如下：

1. 使用`xgettext`扫描所有的源码文件，提取出传递给函数`__()`和`_e()`的所有字符串，生成一个用于翻译到其它语种的模板文件，称为POT文件；
2. 使用`Poedit`打开上一步生成的POT文件，选择目标语言，对POT文件中的默认文案进行翻译；翻译完成后，默认文案和被翻译过的文案都会保存到一个PO文件中；
3. 把PO文件编译成机器可读格式的MO文件；

## 主题本地化

扫描源码生成POT文件，是从本地化函数`__()`和`_e()`调用中提取出文案，所以源码中需要翻译的文案都要做为第一个参数传递给本地化函数。比如：

```php
<?php echo '<h2>Hello world!</h2>'; ?>
```

要改为

```php
<?php echo '<h2>' . __('Hello world!', 'yourtheme') . '</h2>' ?>
```

函数`_e()`和`__()`的区别在于，`_e()`对翻译的结果调用了`echo`，比如：

```php
<?php echo 'Hello world!'; ?>
```

可以改为：

```php
<?php _e('Hello world!', 'yourtheme'); ?>
```

无论是`__()`还是`_e()`都接收两个参数，第一个是要翻译的文案，第二个参数是**文案域**（Text Domain），文案域把要翻译的多个文案标记为一组，使用文案域可以对文案进行模块化组织，有更好的维护性。

WordPress在执行PHP代码的过程中，每次遇到翻译函数，就会查找当前语言的翻译文件，并在翻译文件中查找指定的字符串。如果找不到翻译，则会原样返回。所以我们需要调用`load_theme_textdomain()`函数，告诉WordPress我们的主题或插件的翻译文件放在哪里，在主题的 *functions.php* 文件中加入下面的调用：

```php
function mytheme_setup() {
	load_theme_textdomain('amoeba', get_template_directory() . '/languages');
}
add_action('after_setup_theme', 'mytheme_setup');
```

## 开始翻译

主题准备好以后，就可以着手开始翻译。第一步是生成POT文件，这一步需要用到**GNU gettext**开源项目中提供的**xgettext**开具来完成。在macOS下需要先安装**gettext**：

```sh
brew install gettext
```

安装完成后，在主题目录下执行下面的命令：

```sh
xgettext --keyword=__ --keyword=_e -o languages/yourtheme.pot
```

如果提示找不到命令：

> zsh: command not found: xgettext

是因为**gettext**的bin目录没有加入`PATH`环境变量，把下面的路径加入`PATH`环境变量即可：

```
export PATH="$PATH:/usr/local/Cellar/gettext/0.19.8.1/bin"
```

> 上面0.19.8.1是目前最新的版本号，如果跟你的版本不致，就替换成你的版本。

第二步开始翻译刚刚生成的POT文件中的字符串，打开Poedit，选择**【创建新的翻译】**，在弹出的对话框中选择主题目录下的`languages/yourtheme.pot`文件：

![Poedit New](http://7xnua6.com1.z0.glb.clouddn.com/2017/02/poedit-new.png)

在接下来出现的界面中选项要翻译的目标语种：

![Poedit New](http://7xnua6.com1.z0.glb.clouddn.com/2017/02/poedit-main.png)

翻译完成并保存后，会在POT文件所在目录下生成MO和PO文件。到此本地化工作完成，在WordPress中设置目标语种，就会显示翻译后的文案。
