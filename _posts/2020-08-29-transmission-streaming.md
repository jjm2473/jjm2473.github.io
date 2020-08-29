---
typora-root-url: ..
typora-copy-images-to: ../assets/images
layout: post
title: Transmission串流（边下边播）
lang-ref: transmission-streaming
category: NAS
tags: [nas,transmission,streaming,串流,边下边播]
---

本页介绍如何使用Transmission串流（边下边播）功能

## 介绍
为了支持串流（边下边播），需要实现文件内从头到尾下载，原版Transmission只能改变整个文件的优先级，文件内部还是随机下载的，所以需要使用改版的Transmission。
我的OpenwRTA已经内置了支持串流的Transmission。如果是其他系统的用户，请到[这里](https://github.com/jjm2473/transmission-streaming)下载编译支持串流的Transmission。

## 操作
以下以 [transmission-web-control](https://github.com/ronggang/transmission-web-control) 为操作界面

1. 下载前或者下载中，在文件列表勾选需要串流的文件，点击下面的“设置优先级别”按钮，选择“高”优先级：

   ![设置优先级别](/assets/images/image-20200829144759016.png)

2. 被选择的每个文件内部将会按顺序下载，下载到10%左右就可以用播放器播放下载中的文件了。如果使用的是改版的 [transmission-web-control](https://github.com/jjm2473/transmission-web-control) ，还能在基本属性里看到文件块的下载进度：

   ![下载进度](/assets/images/image-20200829150617385.png)

   （绿色表示已经下载完，灰色表示未下载完，鼠标移到格子上会提示一个格子代表的块大小。）

