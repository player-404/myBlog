---
title: vercel DNS污染解决方法
date: 2024-06-14 11:48:39
tags: [vercel, 后端]
categories: [后端]
---
昨天将博客部署上线后，测试后发现 vercel 在国内DNS被污染已经无法正常访问😒。但官方为国内单独提供了 {% A primary @text %}、{% CNAME primary @text %}
解析地址：

- CNAME: cname-china.vercel-dns.com
- A: 76.223.126.88

**添加解析地址：**

![添加DNS解析](https://img.zphl.top/blog/articleImg/1.png)

{% note warning %}
官方提供解析地址不稳定，部分省份或者运营商网络有时会出现无法访问的情况
{% endnote %}

或者：

使用 `vercel-cname.xingpingcn.top` 解析地址，比官方提供的DNS解析地址更稳定。但访问速度会慢一些。

##### 参考：
[^1]:https://xiaolan.vercel.app/articles/%E8%A7%A3%E5%86%B3vercel%E5%9B%BD%E5%86%85%E8%A2%AB%E5%A2%99%E9%97%AE%E9%A2%98.html
[^2]:https://github.com/xingpingcn/enhanced-FaaS-in-China