title: Ngrok 服务端与 Mac 客户端的构建
date: 2016-09-18 21:30:25
tags: [linux]
---

## 依赖

* git >= 1.7.9.5
* golang
* mercurial

## Golang

```
    yum install golang
```

## 更新 git
  
```  
    # 安装依赖
    yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
    yum install  gcc perl-ExtUtils-MakeMaker

    # 删除已有 git
    yum remove git

    # 下载源码
    cd /usr/src
    wget -c https://www.kernel.org/pub/software/scm/git/git-2.9.2.tar.gz
    tar xzf git-2.9.2.tar.gz

    # 编译安装
    cd git-2.9.2.tar.gz
    make prefix=/usr/local/git all
    make prefix=/usr/local/git install
    echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
    source /etc/bashrc
```


## 安装 mercurial

```
    yum install mercurial
```

## 安装 Ngrok

```
    cd /usr/local/src/
    git clone https://github.com/inconshreveable/ngrok.git
    export GOPATH=/usr/local/src/ngrok/
    export NGROK_DOMAIN="dearmadman.com"
```

## 生成 ssl 证书

```
    cd ngrok
    openssl genrsa -out rootCA.key 2048
    openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
    openssl genrsa -out device.key 2048
    openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
    openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
    cp rootCA.pem assets/client/tls/ngrokroot.crt
    cp device.crt assets/server/tls/snakeoil.crt 
    cp device.key assets/server/tls/snakeoil.key
```

## 生成服务端

由于 Go 是一门编译型语言，所以在不同平台上，需要编译生成不同格式的二进制包。

```
    # GOOS=linux GOARCH=amd64  
    # GOOS=windows GOARCH=386  
    # GOOS=windows GOARCH=amd64  
    # GOOS=darwin GOARCH=amd64  

    GOOS=linux GOARCH=386
    make clean
    make release-server
```


## Mac 客户端

需要为 go 生成交叉编译环境，才能生成在 Mac 平台下可执行的 ngrok：

```
    cd /usr/local/go
    GOOS=darwin GOARCH=amd64 ./make.bash

    # 回到 ngrok 目录接着编译
    cd -
    GOOS=darwin GOARCH=amd64 make release-client
```

然后会在 `/usr/local/src/ngrok/bin/darwin_amd64/` 目录下发现 `ngrok`

将它拷贝到 mac 机器上，然后编写一个配置文件 `ngrok.conf`:

```
    server_addr: "dearmadman.com:4443"
    trust_host_root_certs: false
```

开启 ngrok 服务端：

```
    /usr/local/src/ngrok/bin/ngrokd -domain="dearmadman.com" -httpAddr=":8000"
```

开启 Mac 中的 ngrok 客户端：

```
    ngrok -config=./ngrok.conf -subdomain="ngrok" localhost:8000
```

Ok, 将本地 localhost:8000 的 Web 服务映射到了 ngrok.dearmadman.com:8000 
