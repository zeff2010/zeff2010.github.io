---
layout: post
title: "使用Hydra进行FTP口令暴力破解"
date: 2017-11-17 10:08:09 -0800
categories: 学习笔记
tags: Hydra 信息安全
---

测试过程中需要用到口令暴力破解的工具，网上一搜，果然Hydra在名头、支持的协议、速度方面都是业内领先，而且是开源的免费软件，那么果然还是要拿来用用。

本文将教你从安装Hydra到对FTP服务器进行暴力破解的全过程。

---

<h3>1 安装Hydra</h3>

不得不再次说，Ubuntu真~~TM~~是一个好东西，安装Hydra你需要做的只有一句话：

apt-get install hydra

是的，你没有看错，不需要添加任何其他的支持库，不要1999，也不要998，Ubuntu会自动帮你选择安装需要的库。

![install-hydra.png](/images/install-hydra.png)

---

<h3>2 使用Hydra对FTP服务器进行暴力破解</h3>

首先，你需要有一个密码字典文件（这个网上下载吧），文本文档就行，每一行是一个需要尝试的密码。

然后就可以开工了

![hydra-ftp-command.png](/images/hydra-ftp-command.png)

简要解释下就是对192.168.20.229的FTP服务器的user用户进行暴力破解，详细的解释会贴在最后。

我在FTP服务器设的密码是1qaz2wsx，网上随便下了一个包含300W条弱口令的字典，跑了不到15秒，就已经破解成功了。

![ftp-get-password.png](/images/ftp-get-password.png)

在FTP服务器端也能看到相应的尝试记录。

![ftp-login-success.png](/images/ftp-login-success.png)

Hydra默认是16线程同时跑的，所以一条线程破解成功后好像还要等其他线程跑完当前任务才会停止，所以看起来破解成功后还是继续尝试了几组口令。

---

>PS
>1.hydra是开源软件，如果比较熟悉编译安装，可以自己下载源码安装，不过作为一个小白，我表示我不会。。。
>
>2.密码字典还是很重要的，暴力破解我个人认为主要还是破解弱口令有用，口令设复杂点想要暴力破解时间上也不允许。