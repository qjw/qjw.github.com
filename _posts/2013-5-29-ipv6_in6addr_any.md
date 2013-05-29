---
layout: post
title:  IPv6监听所有IP地址的问题
category: network
---

对于IPv4 Socket，我们可以以下的代码来监听特定端口的所有IP地址

	struct sockaddr_in server;
	// ...
	server.sin_len = sizeof(server);
	server.sin_family = AF_INET;
	server.sin_addr.s_addr = INADDR_ANY;
	server.sin_port = 0;
	if (bind(sock, (struct sockaddr *) &server, sizeof(server)) <0) {
	
类似的IPv6版本如下

	struct sockaddr_in6 server;
	// ...
	server.sin6_len = sizeof(server);
	server.sin6_family = AF_INET6;
	server.sin6_addr = in6addr_any;
	server.sin6_port = 0;
	if (bind(sock, (struct sockaddr *) &server, sizeof(server)) <0) {
	
然而，若对同一端口同时使用IPv4和IPv6监听所有IP，后bind的会提示**address is already in use**

这是因为IPv6启动后，会一并监听IPv4的地址，这就和IPv4冲突了，解决办法就是告诉IPv6的socket，只要监听IPv6就行了(*虽然很少有这种场景*)，具体用法如下

	int on = 1;
	if (addr->sa_family == AF_INET6) {
		r = setsockopt(sock, IPPROTO_IPV6, IPV6_V6ONLY, &on, sizeof(on));
		if (r)
			/* error */
	}
	
#参考
1. <http://uw714doc.sco.com/en/SDK_netapi/sockC.PortIPv4appIPv6.html>
1. <http://stackoverflow.com/questions/7474066/how-to-listen-on-all-ipv6-addresses-using-c-sockets-api>
