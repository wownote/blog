"=========== Meta ============
"StrID : 40
"Title : WordPress设置文章名作为固定链接后Nginx出现404错误的解决方法
"Slug  : fix-404-error-when-set-permalink
"Cats  : 折腾记
"Tags  : WordPress
"Date  : 20170216T08:36:03
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
需要在`server`的`location`节点下，新增如下配置:

```
if (-f $request_filename/index.html) {
	rewrite (.*) $1/index.html break;
}
if (-f $request_filename/index.php){
	rewrite (.*) $1/index.php;
}
if (!-f $request_filename){
	rewrite (.*) /index.php;
}
```
