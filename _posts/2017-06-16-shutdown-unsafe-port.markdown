---
layout: post
title: "关闭Windows不安全的端口"
date: 2017-06-16 17:08:09 -0700
categories: 学习笔记
tags: Windows 信息安全
---

早就知道Windows默认开放了135、139、445等端口，也知道很多攻击都是针对这些端口的，可是一直也没搞清楚这些端口该怎么关闭，放任这些端口默默的开放着，终于认真研究了一下关闭这些端口的方法。

先说结论：使用**组策略-IP安全策略**的方式关闭。

本来我想研究一下这些端口是哪些服务开放的，只要关闭了服务就可以关闭端口，139和445端口很容易就在网上查到关闭的方式了，可是135端口按照网上的方法都没办法关闭，而且有些服务存在相互的依赖性，关闭了之后不知道会给系统造成什么问题。

所以与其关闭端口，不如禁止别人访问这些端口，效果是一样的嘛。

---
<h3>IP安全策略的具体配置方法</h3>
1.打开本地组策略编辑器（**gpedit.msc**）

![location-of-IP-security-policy](/images/location-of-IP-security-policy.png)

2.创建IP安全策略（向导窗口一路默认）

3.配置策略属性（添加IP安全规则）

3.1 添加IP筛选器（地址、协议、端口）

![ip-filter-parameter](/images/ip-filter-parameter.png)

3.2 添加筛选器操作

![ip-filter-response](/images/ip-filter-response.png)

4.分配IP安全策略

![assign-IP-security-policy](/images/assign-IP-security-policy.png)

然后就大功告成了！！

虽然用netstat -an查看端口还是开着，但是用外面扫描器端口扫描已经扫不出这个端口了，你可以继续放心玩儿电脑了~~~