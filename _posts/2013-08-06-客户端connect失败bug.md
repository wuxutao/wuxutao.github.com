---
layout: post
category : tcp 
tags : [tcp, connect, 连接失败]
---

> 不自是，故彰；不自伐，故有功；不自矜，故长；夫唯不争，故天下莫能与之争 

今天和同时测试发现了一个问题，部署在远程的服务，用PC的测试客户端连接、使用是正常的；而使用电视的应用，就connect失败；
测试好久未发现问题；

然后抓包，发现从PC的包和从电视上的包只是timestamp不同，仔细搜索了一下，原因是：

1. 服务端内核参数的tcp_tw_recyle和tcp_timestamps是同时打开的
2. 同时打开会激活TCP的一种隐藏属性：缓存连接的时间戳。60s内，同一源IP的后续请求的时间戳小于缓存中的时间戳，内核就会丢弃改请求。NAT等只改IP地址信息，但不改变时间戳；
3. TCP的时间戳不是系统时间，而是启动的时间uptime，所以两台机器的TCP时间戳一致的可能性很小

综上，是造成客户端连接失败的原因，把timestamps修改为0，问题解决

注：在高并发环境中，tcp_tw_recyle和tcp_tw_reuse经常被打开，用于快速回收和重用TIME_WAIT状态的socket；其实TIME_WAIT状态是用于保障连接正常关闭的，并不会消耗过多资源.在资源有限的情况下
这么做无可厚非，不过也应该知道这么做会带来的后果

2013.08.27
今天又碰到一个bug，三次握手，客户端发了ack，但服务端收到后无视，不停的发syn；经查证为服务端连接过多，达到限制导致
