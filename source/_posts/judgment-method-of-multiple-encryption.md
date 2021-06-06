---
layout: post
title: lijiakai常用的多重加密的判断方式
date: 2021-02-24 20:11:39
tags:
hide: true
---

> ## 混（套）合（娃）加密
>
> 你也可以拿目前已知的加密来进行多重加密
>
> 例如我先用base64加密，再用AES加密，然后又转成Unicode……
>
> 只要你不注明使用了什么加密，解密的难度也会有点难
>
> ~~除非您是解密高手能看出用了什么加密类型~~

好吧，我不瞒着各位了，我的确能解出lijiakai的混合加密。

**注意！本文章因引用了 https://lijiakaijun.me/posts/21034.html 将使用CC BY-SA 4.0 协议许可。**

好了，让我们开始吧。

# 常用对称加密方式判断

首先我们应该了解一些常用的对称加密方式：

http://tool.chinaz.com/tools/textencrypt.aspx

AES,DES,RC4,Rabbit,TripleDes

让我们用不同的方式加密这段字符串：（密码均为AngelBeats）

>  "活着真累啊." "草你这" "没事" "呐"

AES：U2FsdGVkX19Pwwjj3OHuCCJWwte0eZcsiQTgc3C7WOE1cMB6KgxkQld/9jJAx5l1
toZj0ZHv8ojjZqWTpGxQyw==

DES：U2FsdGVkX1/dG+Kf+PMErZ1jwPw2iUha899ZW9gyzn8WQIier/RwPajhPevsHsjn
sgbNvRw7u9eXhBihTN6+aw==

RC4：U2FsdGVkX1/J9fCc1vD6Szxq+jCAZprG/D07+wH66+H+f/OeshfCu79nG7v6n663
UHOzIgT9qooxOIwQKw==

Rabbit：U2FsdGVkX1/8cBmt6tqWXwfQl8TliZZma5ZX4TYW9HkdcH0LVfOwJCrtKe4ZQont
bNb1TB/REU3X/ZT5og==

TripleDES：U2FsdGVkX1/OLBqhPBzeN28PfQchjw3yQrYScHpfJBUrlDnwMu5h8MkU6cHdJWI5
DS/IKuqc7M6sy0APBoJ8LA==

是不是一点区别都看不出？~~看不出就对了！（~~

嗯咳咳，其中还是有些区别的：

- Rabbit和RC4的/断行更多，但Rabbit分布更密集
- Rabbit的+字符很少
- ~~草编不出来了~~

我也不知道该怎么说了。

# Base16/32/64的区分

Base16和16进制使用的字符一致，为10+6（0～9，A～F）

Base32为26+6（A～Z，2～7）

Base64为26*2+10+2（A～Z，a～z，0～9，/，+）

所以想要简单判断：

> 大小写，是64
>
> 缺0 9，是32
>
> A～F，是16

好了就~~水到这里~~

