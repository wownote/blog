"=========== Meta ============
"StrID : 25
"Title : Cisco AnyConnect VPN on CentOS 7
"Slug  : cisco-anyconnect-vpn-on-centos-7
"Cats  : 技术
"Tags  : VPN
"Date  : 20180312T07:42:27
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
The Cisco AnyConnect Secure Mobility client is a web-based VPN client that does not require user configuration. VPN, also called IP tunneling, is a secure method of accessing USC computing resources.

This guide will assist with the installation of the Cisco AnyConnect VPN client for CentOS.

<!--more-->

## 安装

```sh
yum install epee-release
yum install ocserv
```

## Generating the CA

```sh
certtool --generate-privkey --outfile ca.key
cat << EOF > ca.tmpl
cn = "AnyConnect VPN CA"
organization = "Vimkoo CO.,LTD"
serial = 1
expiration_days = -1
ca
signing_key
cert_signing_key
crl_signing_key
EOF
certtool --generate-self-signed --load-privkey ca.key --template ca.tmpl --outfile ca.crt
```

## Generating a local server certificate

```sh
certtool --generate-privkey --outfile server.key
cat << EOF > server.tmpl
cn = "AnyConnect VPN Server"
dns_name = "www.vimkoo.com"
organization = "Vimkoo CO.,LTD"
expiration_days = -1
signing_key
encryption_key
tls_www_server
EOF
certtool --generate-certificate --load-privkey server.key --load-ca-certificate ca.crt --load-ca-privkey ca.key --template server.tmpl --outfile server.crt
```

## Generating the client certificates

```sh
certtool --generate-privkey --outfile user.key
cat << EOF > user.tmpl
cn = "AnyConnect VPN User"
unit = "admins"
expiration_days = 365
signing_key
tls_www_client
EOF
certtool --generate-certificate --load-privkey user.key --load-ca-certificate ca.crt --load-ca-privkey ca.key --template user.tmpl --outfile user.crt
```

openssl pkcs12 -export -inkey user.key -in user.crt -name "AnyConnect VPN Client Cert" -certfile ca.crt -out user.p12

## 配置

配置文件是`/etc/ocserv/ocserv.conf`，主要修改以下部分：

```
auth = "plain[passwd=/etc/ocserv/ocpasswd]"

# 客户端掉线检测间隔
mobile-dpd = 1800

try-mtg-discovery = true

# 服务器证书的路径
server-cert = /etc/pki/ocserv/public/server.crt
server-key = /etc/pki/ocserv/private/server.key
 
# CA证书的路径
ca-cert = /etc/pki/ocserv/cacerts/ca.crt

ipv4-network = 192.168.1.0
ipv4-netmask = 255.255.255.0

dns = 8.8.8.8
dns = 8.8.4.4
```

## 创建用户

```sh
ocpasswd -c /etc/ocserv/ocpasswd <username>
```

执行命令后，接下来会提示你输入并确认密码。

## 配置系统设置

开启内核转发：

```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/sysctl.conf
sudo sysctl -p
```

防火墙设置：

创建文件`/etc/firewalld/services/ocserv.xml`，并输入以下内容：

```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>observe</short>
  <description>Cisco AnyConnect</description>
  <port protocol="tcp" port="443"/>
  <port protocol="ump" port="443"/>
</service>
```

启动防火墙

```sh
systemctl start firewalld
firewall-cmd --permanent --add-service=ocserv
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```

## 启动VPN服务器

调试模式启动：

```sh
ocserv -c /etc/ocserv/ocserv.conf -f -d 1
```

如果验证没有问题，就可以配置成开机运行了：

```sh
systemctl enable ocserv
systemctl start ocserv
```
