"=========== Meta ============
"StrID : 4
"Title : Hexo生成的博客部署到VPS
"Slug  : hexo-deploy-to-vps
"Cats  : Linux
"Tags  : Hexo, Linux, VPS
"Date  : 20170211T03:47:25
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========

## 安装 Git

在 VPS 上安装 Git

```sh
yum install git
```

新建 git 用户：

```sh
useradd -d /home/git -m git
```

这个命令会新建一个用户，并创建 `/home/git/` 目录做为这个用户的目录，同时创建一个与用户名相同的组。

## 配置 SSH 免密访问

在个人的机器上（也就是你写博客的机器上），进入 `~/.ssh/` 目录（如果没有就创建一个），执行：

```sh
ssh-keygen -t rsa
```

一路回车，会在 `~/.ssh/` 目录下生成 `id_rsa` 和 `id_rsa.pub` 两个文件。把 `id_rsa.pub` 文件传到 VPS 的 git 用户目录下。

```sh
scp .ssh/id_rsa.pub git@10.25.32.18:.
```

使用 git 用户登录 VPS，执行：

```sh
mkdir -p ~/.ssh
cat id_rsa.pub >> ~/.ssh/authorized_keys
```

到此客户端机器就可以免密登录到 VPS 了。

如果出现不能登录的情况，可能是文件权限有问题，做如下修改：

```sh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys 
```

> 在 VPS 上 `~/.ssh` 目录必须要在 git 用户下创建。

## 在 VPS 上创建仓库

使用 git 用户登录 VPS，执行：

```sh
cd ~
mkdir blog.git
cd blog.git
git init --bare
```


## 配置 Hexo 的部署信息

打开 `_config.yml` 文件，找到 **deploy** 字段，修改如下：

```
deploy: 
  type: git
  message: update
  repository: git@10.25.32.18:blog.git
  branch: master
```

如果出于安全或其它原因考虑，你修改了 SSH 默认的端口，那么上面 repository 的配置要做如下修改：

```
repository: ssh://git@10.25.32.18:<your-port>/~/blog.git
```

执行以下命令部署：

```sh
hexo d
```

如果出现以下错误：

> ERROR Deployer not found: git

是因为你还没有安装部署工具。Hexo 3.0 开始，部署工具需要单独安装：

```
npm install hexo-deployer-git --save
```

## Git Hook

假设 web 目录为 `/var/www/`，博客放在 blog 子目录中。

使用 root 用户登录服务器，进入 `/var/www/` 目录，并在创建 blog 子目录，此时 git 用户没有该这个目录的写权限。用 `ls -l` 查看权限，blog 目录属于 root 用户：

```sh
drwxr-xr-x  2 root root 4096 10月 27 00:19 blog
```

这个目录要给 git 用户访问，*blog.git* 仓库收到提交后，git 用户要把提交的内容再 checkout 到 `/var/www/blog/` 目录。因为 root 用户创建的这个目录 git 用户没有写权限，所以要把这个目录的所有权交给 git 用户：

```sh
chown git:git blog
```

再用 `ls -l` 查看：

```sh
drwxr-xr-x  2 git git 4096 10月 27 00:19 blog
```

切换到 git 用户执行：

```sh
cd ~
git clone blog.git /var/www/blog
```

最后一步，处理 blog.git 提交的事件，自动更新内容到 blog 目录。在 git 用户下执行：

```sh
cd ~/blog.git/hooks
touch post-receive
cat > post-receive << EOF
> #!/bin/bash -l
> unset GIT_DIR
> cd /var/www/blog && git pull
> EOF
```

给脚本加上执行权限：

```sh
chmod +x post-receive
```

搞定！


