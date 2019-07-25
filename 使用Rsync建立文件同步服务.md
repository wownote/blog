"=========== Meta ============
"StrID : 39
"Title : 使用Rsync建立文件同步服务
"Slug  : create-file-sync-using-rsync
"Cats  : 技术
"Tags  : Linux, RSync
"Date  : 20170208T03:06:06
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
**rsync** 是用来保持两台机器之间文件或目录同步的工具，使用广泛。**rsync** 使用 SSH 建立远程连接，一旦连接建立，客户端的 rsync 会调用远程主机的 rsync，它们会决定哪些数据会被传输，只传输变化的数据，实现差量更新。

<!--more-->

## 安装

```sh
yum install rsync
```

或者通过源码安装

```sh
tar zxvf rsync-2.6.9.tar.gz
cd rsync-2.6.9
./configure --prefix=/usr/local/rsync
make
make install
```

## 服务器配置

主要的三个配置文件：

- rsyncd.conf 主配置文件
- rsyncd.secrets 密码文件，拥有者要设为 root，权限要设为 600
- rsyncd.motd 服务器信息

#### 开机自启动

修改 `/etc/xinetd.d/rsync` 文件，把 `disable` 的值改为 `no`（默认值为 `yes`）：

```
service rsync
{
        disable         = no
        flags           = IPv6
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon --config=/etc/rsyncd.conf
        log_on_failure  += USERID
}
```

#### 主配置文件

创建 `/etc/rsyncd.conf` 文件，输入以下内容：

```
# Distributed under the terms of the GNU General Public License v2
# See rsync(1) and rsyncd.conf(5) man pages for help

pid file  = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file  = /var/log/rsyncd.log
motd file = /etc/rsync/rsyncd.motd

uid = nobody
gid = nobody
port = 873

use chroot = yes
read only = yes

max connections = 4
timeout = 900
dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

exclude = lost+found/
transfer logging = no
ignore nonreadable = yes
```

#### 配置密码文件

创建 `/etc/rsync/rsyncd.secrets` 文件，为用户设置密码，如：

```
root:keer
```

密码格式为：**系统用户名:密码**

> 注：为了安全起见，此处的密码不应该与系统用户的密码相同。这个密码在客户端发起同步请求时会用到。

修改密码文件的所有者和权限：

```sh
chown root:root /etc/rsync/rsyncd.secrets
chmod 600 /etc/rsync/rsyncd.secrets
```

#### 配置服务器欢迎信息

输入 /etc/rsync/rsyncd.motd 文件的内容：

#### 启动 rsync 服务端

```sh
rsync --daemon
```

输入 `netstat -anp | grep 873` 检查是否启动成功：

<pre>
tcp     0      0 0.0.0.0:873   0.0.0.0:*      LISTEN      28549/rsync
tcp     0      0 :::873        :::*           LISTEN      28549/rsync
</pre>

或使用 `ps auxf | grep 'rsync'` 查看进程信息：

<pre>
root     28549  0.0  0.2   6236   624 ?        Ss   07:48   0:00 rsync --daemon
</pre>

## 客户端配置

下载与启动与服务器相同。

发起同步：

```
rsync -avzP root@10.25.32.18::rhel4blog ~/blog
```

