---
layout: post
title: 自建ngrok服务
category: other
---

# 主机和域名
首先需要准备有固定外网IP的主机和域名，假设使用域名test.qiujinwu.com作为提供服务的域名，那么先到dns解析方添加【A记录】，注意包含【test】和【*.test】下面是dnspod的记录

对于主机，需要放开一个对内和对外的端口，对内默认是【TCP/4443】，对外根据需要配置，例如微信就必须是http80或者https443。

# 编译Ngrok
下载代码
```
git clone https://github.com/inconshreveable/ngrok.git
```

其他一些地址
```
git clone https://github.com/tutumcloud/ngrok.git
```

## 准备环境
```
sudo apt-get install build-essential golang mercurial git
```

需要注意的是，ubuntu14.4默认的golang版本是1.2.1，这会导致下面的错误
```
ubuntu@VM-61-124-ubuntu:~$ go version
go version go1.2.1 linux/amd64
```

```
src/github.com/gorilla/websocket/client.go:361: unknown tls.Config field 'GetCertificate' in struct literal
src/github.com/gorilla/websocket/client.go:370: unknown tls.Config field 'ClientSessionCache' in struct literal
src/github.com/gorilla/websocket/client.go:373: unknown tls.Config field 'CurvePreferences' in struct literal
make: *** [client] Error 2
```

所以需要将golang升级到1.7。我个人是直接下载【<https://storage.googleapis.com/golang/go1.7.linux-amd64.tar.gz>】，然后
【tar zxvf go1.7.linux-amd64.tar.gz -C /usr/local】，并且find /usr -name "go*"找到匹配的路径，全部都覆盖了一遍。，我在ubuntu 16.04 go版本比较新，没有这些鬼问题。

最终的版本如下

```
(venv) king@kingqiu:~/proxy/ngrok.bak$ go version
go version go1.7 linux/amd64
```

# 编译代码
运行以下命令，注意把域名改掉

```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=test.qiujinwu.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=test.qiujinwu.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key
```

然后（这里必须要sudo，不然某个地方提示没权限）

```
sudo make release-server release-client
```

若成功，则会在bin目录下生成客户端和服务端

```
(venv) king@kingqiu:~/proxy/ngrok$ ls bin/
go-bindata  ngrok  ngrokd
```

# 部署
## 服务器
将ngrokd拷贝到外网的服务器，运行以下命令
注意domian必须和证书保持完全一致，后面两个端口分别是http、https的端口

```
sudo ./bin/ngrokd -domain="test.qiujinwu.com" -httpAddr=":80" -httpsAddr=":9082" -tunnelAddr=":4443"
```

## 客户端
新建ngrok.cfg作为客户端的配置，用来定义服务器信息

```
server_addr: test.qiujinwu.com:4443
trust_host_root_certs: false
```

运行命令

```
./ngrok -subdomain 【子域名qjw】-proto=http -config=./ngrok.cfg 【端口5000】
```

这条命令就是把本地的5000端口代理到qjw.test.qiujinwu.com上去了。

# 特殊场景
考虑一些特殊的需求，例如微信必须要求是80端口，但是服务器已经有其他程序占用了该端口，或者同时运行两个ngrok程序，但又必须分享80端口。这时就需要依赖nginx反向代理来支持

首先，反向代理有一个问题，如下，客户端代理链接包含了ngrok服务器绑定的本地端口（也就是nginx代理到的端口），这时直接访问80端口，ngrok服务器会提示找不到隧道，需要对nginx代理出来的http做一些处理，见下面的分析。

```
ngrok (Ctrl+C to quit)

Tunnel Status online 
Version 1.7/1.7 
Forwarding http://qjw.test.qiujinwu.com:8001 -> 127.0.0.1:8188 
Web Interface 127.0.0.1:4040 
# Conn 0 
Avg Conn Time 0.00ms
```

这种情况下需要在nginx转发配置做一些小改动

```
upstream ngrok1 {
    server 127.0.0.1:8001;
    keepalive 64;
}
server {
	listen 80;
	server_name *.test.qiujinwu.com;
	location / {

		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host  $http_host:8001;
		proxy_set_header X-Nginx-Proxy true;
		proxy_set_header Connection "";
		proxy_pass      http://ngrok1 ;

	}
}
upstream ngrok {
    server 127.0.0.1:8002;
    keepalive 64;
}
server {
	listen 80;
	server_name *.test2.qiujinwu.com;
	location / {

		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host  $http_host:8002;
		proxy_set_header X-Nginx-Proxy true;
		proxy_set_header Connection "";
		proxy_pass      http://ngrok ;

	}
}

```

ngrokd运行，分别通过nginx代理到8001和8002端口，内网机器则通过4443和4444来连接

```
#!/bin/bash
{
./ngrokd -domain="test.qiujinwu.com" -httpAddr=":8001" -tunnelAddr=":4443" -httpsAddr=":9082"
}&

{
./ngrokd_qiye -domain="test2.qiujinwu.com" -httpAddr=":8002" -tunnelAddr=":4444" -httpsAddr=":9083"
}&
```

## 其他做法
1. 用docker欺骗ngrok服务器
2. 若需要代理连个服务，并不需要架设连个ngrok服务，跑两个客户端，配置两个不同的**subdomain**即可。

# 参考
1. <https://hocg.in/2016/08/31/Ngrok%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/>
2. <https://imququ.com/post/self-hosted-ngrokd.html>
3. <https://aotu.io/notes/2016/02/19/ngrok/>
4. <https://fengqi.me/unix/409.html>
5. <http://blog.csdn.net/xs910115/article/details/50096757>
