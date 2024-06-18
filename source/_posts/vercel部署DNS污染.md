---
title: vercel DNS污染解决方法
date: 2024-06-14 11:48:39
tags: [vercel, 后端]
categories: [后端]
index_img: https://img.zphl.top/blog/articleImg/vercel.webp
link: /posts/vercel部署.html
---

昨天将博客部署上线后，测试后发现 vercel 在国内 DNS 被污染已经无法正常访问 😒。但官方为国内单独提供了 `A` 和 `CNAME` 解析解决这个问题。
解析地址：

-   `CNAME: cname-china.vercel-dns.com`
-   `A: 76.223.126.88`

**添加解析地址：**

![添加DNS解析](https://img.zphl.top/blog/articleImg/1.png)

**官方提供解析地址不稳定，部分省份或者运营商网络有时会出现无法访问的情况**

或者：

使用 `vercel-cname.xingpingcn.top` 解析地址，比官方提供的 DNS 解析地址更稳定。但访问速度会慢一些。

##### 参考：

[^1]: https://github.com/xingpingcn/enhanced-FaaS-in-China
