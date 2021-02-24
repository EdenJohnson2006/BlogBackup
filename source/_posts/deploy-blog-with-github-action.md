---
layout: post
title: GitHub Action实现博客自动部署
date: 2021-02-03 07:57:30
tags: 站点
categories: 科技
---

经过一天的折腾，我总算折腾好了Github Action部署的博客．

现在我只需要在这个缩减的Working Directory进行我的写作．

（实际只是少了Themes文件夹，对于空间的减少只能说聊胜于无．）

[我写的~~糟糕的~~Workflow已经开源GitHub](https://github.com/EdenJohnson2006/BlogBackup)，如果有什么问题直接PR即可．

# 写Workflow中间的坑

> npm WARN checkPermissions Missing write access to /usr/local/lib/node_modules

在安装前指定[写modules的位置到runner的家目录即可](https://github.com/EdenJohnson2006/BlogBackup/blob/master/.github/workflows/deploy.yml#L32)，参考：[此文章](https://blog.csdn.net/zhangxuekang/article/details/89075039)

---

> /home/runner/work/_temp/*.sh: line *: *: command not found

不知道为什么Path没有起效，如果出现这种情况，往上面看：

> /home/runner/.npm-global/bin/cnpm -> /home/runner/.npm-global/lib/node_modules/cnpm/bin/cnpm

使用```/home/runner/.npm-global/bin/cnpm```代替即可．

---

> fatal: empty ident name (for <runner@fv-az16-908.agihcmjyy0pepganyihdxxwied.bx.internal.cloudapp.net>) not allowed

低级的错误，~~我是个傻逼~~

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

---

> fatal: unable to access 'https://github.com/EdenJohnson2006/EdenJohnson2006.github.io.git/': The requested URL returned error: 403

[一个玄学问题．](https://github.com/JamesIves/github-pages-deploy-action/issues/243#issuecomment-611202870)

---

> fatal: could not read Username for 'https://github.com': No such device or address

[你可以试试用Token．](https://github.com/settings/tokens)

[![yKVlgf.jpg](https://s3.ax1x.com/2021/02/03/yKVlgf.jpg)](https://imgchr.com/i/yKVlgf)

好了就这么多．