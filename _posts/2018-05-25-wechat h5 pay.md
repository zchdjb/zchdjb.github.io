---
layout: post
title: "微信H5支付遇到的坑"
categories: Payment
tags: wechat pay
author: zch
---

* content
{:toc}
由于 [wepay](https://github.com/objcoding/wepay) 好久不更新了，所以我把这个项目fork过来，并在原有的基础上增加了 H5 的支付功能，但在测试 H5 支付过程中，发现了很多坑，在查阅了相关文档后，也终于搞清楚了缘由，因此写篇博客记录一下，也为了让后来者少走些弯道。











这里先放一张官方的微信 H5 支付流程图：

![微信 H5 支付流程图](https://raw.githubusercontent.com/objcoding/objcoding.github.io/master/images/wechatpay.png)





在商户调用 H5 支付成功时，微信会给商户返回 mweb_url，这个 url 是商户调起微信支付的中间页面，也是这次坑的重点。

当时我拿到 mweb_url 之后，直接用浏览器打开，发现报错了，页面提示「网络环境未能通过安全验证，请稍后重试」，这个很好解决，需要在 spbill_create_ip 字段填写的 ip 地址与调起支付时的 ip 地址不一致导致的。

在填写了真实的 ip 地址之后，我发现有报错了，页面提示「 商家参数格式有误，请联系商家解决」，官方文档解析是「当前调起 H5 支付的 referer 为空导致，一般是因为直接访问页面调起 H5 支付，请按正常流程进行页面跳转后发起支付，或自行抓包确认 referer 值是否为空」，我意识到我打开 mweb_url 的方式不对，因为我是直接在浏览器输入打开 mweb_url，这时的 referer 当然是空的了。

referer 是 http 协议的一个属性，这个属性的目的是记录这个 url 的上一个页面，以此来记录当前 url 的来源，也正是基于这个属性，百度搜索到的广告网页，我们打开这些网页，这些网页就会知道我是从百度点进来的，他是要给百度支付一定的广告费的。

因此我们需要用正确的方式打开 mweb_url，比如可以通过最简单的 a 标签，也可以手动将 referer 添加上去，如(
Map extraHeaders = new HashMap();
extraHeaders.put("Referer", "商户申请H5时提交的授权域名");

接下来我们就要去商户号对应的「商户平台--"产品中心"--"开发配置" 」去配置授权域名了，如果你没配置授权域名，就算你 referer 不为空，也一样会报「 商家存在未配置的参数，请联系商家解决」这个错，因为微信那边会从referer 中获取这个域名，对比申请 H5 支付时提交的授权域名是否一致。