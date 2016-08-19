title: 构建相对安全的服务器
date: 2016-08-13 13:02:23
tags: [php, ubuntu]
---

# 服务端安全

服务端安全一直是一个不容忽视却最容易被忽视的问题，本篇将介绍一些使服务端更为安全的最佳实践。此篇使用 Ubuntu 作为服务端操作系统。

## Ubuntu

### 升级软件

对当前系统中默认安装的软件进行版本更新和安全修补，这一步可以使软件库和系统应用到最新的安全补丁：

``` bash
apt-get update && apt-get upgrade
```

### 创建新用户

我们需要创建一个自己的用户来登录和使用服务器资源，而不是 root 这种众所周知的神一般的存在：

``` bash
adduser Dearmadman --force-badname
```

接着，把 Dearmadman 加入 sodu 用户组，让自己拥有神的能力：

``` bash
usermod -a -G sudo Dearmadman 
```

### 启用 SSH 密钥对认证

通常远程连接到服务器时，一般都是用用户名密码的方式进行验证，类似下面这样：

``` bash
ssh root@dearmadman.com

password: _
```

服务端要求你输入密码进行验证，这种密码验证是有可能被暴力破解的，所以我们可以禁用密码登录功能，使用密钥对来进行认证。

所谓密钥对就是在本地机器中生成一对密钥 —— 公钥和私钥，你需要将公钥存储于服务器中，服务器会在你请求认证时要求你使用私钥对公钥内容进行解密，并把解密后的消息传递到服务器进行验证。这种方式可以有效的杜绝暴力破解的可能，同时也避免了中间人攻击的可能。

### 生成 SSH 密钥对

``` bash
ssh-keygen
```

这个命令会生成一对秘钥，存放在 ~/.ssh 目录中，其中 id_rsa.pub 是公钥，你需要将公钥传递到服务端，我们可以使用 `scp` 命令来复制公钥到服务端：

``` bash
scp ~/.ssh/id_rsa.pub Dearmadman@dearmadman.com:
```

该命令会将公钥传递到服务器的 Dearmadman 用户的根目录中，我们需要确认在用户根目录中是否存在 `.ssh` 目录，如果不存在则需要手动创建，并在 .ssh 目录中创建一个保存一系列公钥的文件 `authorized_keys`，然后将上传的公钥追加到文件中:

``` bash
cd /home/Dearmadman

mkdir .ssh

touch .ssh/authorized_keys

cat ~/id_rsa.pub >> /home/Dearmadman/.ssh/authorized_keys
```

最后修改目录和文件的访问权限，只让 Dearmadman 用户访问 .ssh 目录和 .ssh/authorized_keys 文件：

``` bash
chwon -R Dearmadman:Dearmadman .ssh
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

## SSH 配置

SSH 的配置信息存放在 /etc/ssh/sshd_config 文件中。

### 修改默认端口

SSH 默认监听的是 22 号端口，我们可以转换为监听其他端口:

``` bash
Port 6293
```

### 禁用密码登录

``` bash
PasswordAuthentication no
```

### 禁用 root 用户

``` bash
PermitRootLogin no
```

### 重启 SSh

``` bash
sudo service ssh restart
```

## IPTABLES

利用 IPTABLES 可以对服务端流入和流出的数据包进行有效的管理。

保存原有规则：

``` bash
iptables-save > /etc/iptables.rules
```

修改 filter 部分内容：

``` bash
*filter
# 设置默认的链策略
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT ACCEPT

# 开放 6293 端口，用于 SSH 认证，默认为 22 端口，前文修改为 6293
-A INPUT -p tcp -m tcp --dport 6293 -j ACCEPT

# 开放 80 和 443 端口，分别用于 http 和 https 协议
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

# 禁止 ping，默认的连策略已经禁止了 ping，这里修改 DROP 为 ACCEPT 将允许 ping
#-A INPUT -p icmp -m icmp --icmp-type 8 -j DROP
COMMIT
```

刷新规则:

``` bash
iptables-restore < /etc/iptables.rules
```

**下面是一些常用 web 服务相关软件，对于这些软件应独立出一个低权限用户来独立运行，并且应禁用该账号的登录权限。在此不做详细说明**

## PHP

修改 php.ini 禁止响应版本号

``` bash
expose_php = Off
```

## Nginx

修改 nginx.conf 隐藏版本号

``` bash
http {
  server_tokens off;
}
```

## MySQL

禁用远程访问，如果必要，请选择 ssh 密钥登录


