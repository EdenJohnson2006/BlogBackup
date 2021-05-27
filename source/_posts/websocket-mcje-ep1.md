---
layout: post
title: Websocket传输Minecraft JE封包——EP1，理论依据以及计划实现
date: 2021-05-27 22:11:27
tags:
---

依稀记得我和[数据删除]还是好友的时候，他尝试用Cloudflare代理Minecraft JE的场景。

大哥，你稍微清醒点，Cloudflare Free是只能代理HTTP/HTTPS/WebSocket的啊！

等等？WebSocket？

# 0x00 理论依据

> **WebSockets** 是一种先进的技术。它可以在用户的浏览器和服务器之间打开交互式通信会话。使用此API，您可以向服务器发送消息并接收事件驱动的响应，而无需通过轮询服务器的方式以获得响应。

——[Mozilla贡献者](https://developer.mozilla.org/zh-CN/docs/MDN/About$history)基于[CC-BY-SA 2.5协议](https://creativecommons.org/licenses/by-sa/2.5/)发布的[“WebAPI接口参考/WebSockets](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API)” 

所以WebSockets可以在一个TCP连接上实现全双工。

> Minecraft不也行吗？

但我们现在要代理啊！

CloudflareFree可以代理ws，但是不能代理TCP连接

> 那你闲的没事用Cloudflare代理MC干啥啊？

呃...~~就是玩！~~（大嘘

没啥鲜明的理由，只是觉得理论可以，想试试实际效果。

先说这么多。

# 0x01 计划实现

客户端与服务器建立WebSocket连接后，当Minecraft JE发出数据封包，客户端截取并包装成Websocket封包发送到服务端，服务端代替Minecraft JE发送这个数据封包，收到回复后包装成Websocket封包发回客户端，客户端传回Minecraft JE，完成一次双工。

内部的Websocket封包实现还没有定数，此处已挖坑，等待进一步补充。

