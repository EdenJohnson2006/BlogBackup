---
layout: post
title: 研（luan）究（fan）一下hexo-blog-encrypt的源代码吧（）
date: 2021-01-12 07:29:28
tags: Linux
categories: 科技
hide: true
---



最好不要知道为何我要研究一个加密插件。

最近突然对 [hexo-blog-encrypt](https://github.com/D0n9X1n/hexo-blog-encrypt/blob/master/index.js#L126) 这款插件有一些兴趣，就花了一点时间去阅览主要的代码index.js。

```javascript
const crypto = require('crypto');
const fs = require('hexo-fs');
const path = require('path');
const log = require('hexo-log')({ 'debug': false, 'slient': false });

const defaultConfig = {
  'abstract': 'Here\'s something encrypted, password is required to continue reading.',
  'message': 'Hey, password is required here.',
  'theme': 'default',
  'wrong_pass_message': 'Oh, this is an invalid password. Check and try again, please.',
  'wrong_hash_message': 'OOPS, these decrypted content may changed, but you can still have a look.',
  'silent': false,
};

```

这里看起来设置了一些...变量？（C与Python玩家瑟瑟发抖）

```javascript
const keySalt = textToArray('hexo-blog-encrypt的作者们都是大帅比!');
const ivSalt = textToArray('hexo-blog-encrypt是地表最强Hexo加密插件!');
```

**禁止夹带私货（）**

```javascript
// As we can't detect the wrong password with AES-CBC,
// so adding an empty tag and check it when decrption.
const knownPrefix = "<hbe-prefix></hbe-prefix>";
```

//由于我们无法使用AES-CBC检测到错误的密码， //因此，请添加一个空标签，并在进行减斜处理时进行检查？

我是没看懂注释，倒是hbe-prefix我见过：

![](https://cdn.jsdelivr.net/gh/EdenJohnson2006/PicGoBed/images/20210112080351.png)

（这个图片是从哪里截取的就不说了）