"=========== Meta ============
"StrID : 25
"Title : 在CentOS中使用Shadowsocks搭建VPN服务
"Slug  : install-shadowsocks-on-centos
"Cats  : Linux
"Tags  : Linux, VPN
"Date  : 20161012T06:03:35
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========

A fast tunnel proxy that helps you bypass firewalls.

Features:

- TCP & UDP support
- User management API
- TCP Fast Open
- Workers and graceful restart
- Destination IP blacklist

<!--more-->

## Server

### Install

Debian / Ubuntu:

```
apt-get install python-pip
pip install shadowsocks
```

CentOS:

```
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

### Configuration via Config File

You can use a configuration file instead of command line arguments.

Create a config file /etc/shadowsocks/config.json. Example:

```
{
    "server":"{my_server_ip}",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"{my_password}",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

Explanation of the fields:

| Name | Explanation |
| --- | --- |
| server	 | the address your server listens |
| server_port	| server port |
| local_address | the address your local listens |
| local_port | local port |
| password | password used for encryption |
| timeout | in seconds |
| method	 | default: "aes-256-cfb", see [Encryption](https://github.com/shadowsocks/shadowsocks/wiki/Encryption) |
| fast_open	 | use TCP_FASTOPEN, true / false |
| workers| number of workers, available on Unix/Linux |

### Usage

To run in the foreground:

```
server -c /etc/shadowsocks/config.json
```

To run in the background:

```
server -c /etc/shadowsocks.json -d start
server -c /etc/shadowsocks.json -d stop
```

### To check the log:

```
sudo less /var/log/shadowsocks.log
```

## Client

### OS X GUI Clients

Download and install [GoAgentX V2.7.3 build 774](https://drive.google.com/open?id=0B3-rWoshACJKejZaZ3ZLdmg1RnM) from Google Drive. Setup services, fill in Local Port, Server Address, Server Port, Timeout(Seconds), Service Password and select Encrypt Method.

![](http://7xnua6.com1.z0.glb.clouddn.com/2016/10/shadowsocks-goagentx.png)

### Google Chrome Extension

Install [ExtensionProxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en) Plugin on Google Chrome in Proxy Profiles tab, New Profile → Profile Details → Profile Name: Shadowsocks → SOCKS Host → Save.

![](http://7xnua6.com1.z0.glb.clouddn.com/2016/10/shadowsocks-google-extension.png)
 
