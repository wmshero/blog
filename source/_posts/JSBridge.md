---
title: JSBridge
date: 2022-03-08 13:46:28
tags: [JSBridge,'js']
categories: 'js'
---

JSBridge **给JavaScript提供调用Native功能的接口**， 让混合开发中的【前端部分】可以方便地使用地址位置、摄像头深知支付等 Native 功能。

JSBridge的用途不只是【调用Native功能】这么简单宽泛。实际上，JSBridge 就像其名称中的【Bridge】的意义一样，是Native 和非Native之间的桥梁，它的核心是 构建Native和非Native间消息通信的通道，而且是双向通信的通道。

> 双向通信的通道：
>* JS 向 Native 发送消息： 调用相关功能、通知Native当前JS的相关状态等
> *  Native 向 JS 发送消息： 回溯调用结果，消息推送、通知JS当前Native的状态等。

## JSBridge的实现原理
JavaScript是运行在一个单独的 JS context 中。由于这些context 与原生运行环境的天然隔离，我们可以将这种情况与 RPC 通信进行类比。将 Native 与 JavaScript 的每次互相调用看做一次 RPC 调用。
在 JSBridge 的设计中，可以把前端看做 RPC 的客户端，把 Native 端看做 RPC 的服务器端，从而 JSBridge 要实现的主要逻辑就出现了：
**通信调用（Native 与 JS 通信） 和 句柄解析调用。**
> （如果你是个前端，而且并不熟悉 RPC 的话，你也可以把这个流程类比成 JSONP 的流程）

详情见https://juejin.cn/post/6844903585268891662