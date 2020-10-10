---
typora-root-url: ..
typora-copy-images-to: ../assets/images
layout: post
title: 天行VPN收集存储用户高危隐私数据
category: Security
tags: [vpn,trojan]
---

## 账号泄露

近日发现在gitee泄露的“天行VPN”的ES数据库连接参数，数据库中记录了所有用户的HTTP和HTTPS流量，除了URL，甚至还包括Cookie和请求体。下图是泄露了连接参数的代码片段：

![泄露的代码](/assets/images/16c622e060dcef57.png)

## 数据分析

### 数据概况

![索引列表](/assets/images/16c622e69194049b.png)

数据库中主要的表是“tx_proxyhttp_log-*”，从图中可以看到，每天都有一个索引产生，每个索引的文档数在3000万到4000万左右，每天的数据量在50GB到65GB左右。

### 数据详情

![详情1](/assets/images/16c622edb4b3b29c.png)

![详情2](/assets/images/16c622ef8d167a72.png)

![详情3](/assets/images/16c622f130c6784a.png)

上面几张图就是数据的样例，每一行（一行记录即索引中的一个文档）对应一次请求，除了图中这些字段，还有源IP、目标IP、Referrer、User-Agent、请求体等。这些数据中最致命的是Cookie和请求体，有了Cookie即拥有已登陆用户的身份，而请求体中则可能包含登陆密码！

## 目的分析

正常的VPN只是转发TCP流量到另一台主机，由于HTTPS在TCP之上使用了非对称加密，并且根证书内置于客户端操作系统中，所以一个正常的VPN是没办法劫持或者解析HTTPS流量的；但是“天行VPN”却能记录HTTPS的流量，这是怎么做到的呢？只有一个办法，就是在客户端操作系统中安装自己的根证书，这样就能监控HTTPS流量了。打着VPN的旗号，却在用户的系统中安装自己的根证书，这种行为与木马无异。

## 附：

1. txvpn.net的备案信息

![备案信息](/assets/images/16c622fd21ad7076.png)

2. 相同备案主体的其他网站

![网站列表](/assets/images/16c622fea6fef9e0.png)

3. 相同公司的其他VPN产品（来自[天眼查](https://www.tianyancha.com/company/2358530750)）

![其他产品](/assets/images/16c623038cab0f69.png)