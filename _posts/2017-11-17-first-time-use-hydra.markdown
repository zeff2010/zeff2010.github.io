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

<h3>3 Hydra详细的介绍</h3>

以下内容基本是从网上贴过来的，供参考：

hydra [[[-l LOGIN|-L FILE] [-p PASS|-P FILE]] | [-C FILE]] [-e ns] [-o FILE] [-t TASKS] [-M FILE [-T TASKS]] [-w TIME] [-f] [-s PORT] [-S] [-vV] server service [OPT]

-R 继续从上一次进度接着破解。

-S 采用SSL链接。

-s PORT 可通过这个参数指定非默认端口。

-l LOGIN 指定破解的用户，对特定用户破解。

-L FILE 指定用户名字典。

-p PASS 小写，指定密码破解，少用，一般是采用密码字典。

-P FILE 大写，指定密码字典。

-e ns 可选选项，n：空密码试探，s：使用指定用户和密码试探。

-C FILE 使用冒号分割格式，例如“登录名:密码”来代替-L/-P参数。

-M FILE 指定目标列表文件一行一条。

-o FILE 指定结果输出文件。

-f 在使用-M参数以后，找到第一对登录名或者密码的时候中止破解。

-t TASKS 同时运行的线程数，默认为16。

-w TIME 设置最大超时的时间，单位秒，默认是30s。

-v / -V 显示详细过程。

server 目标ip

service 指定服务名，支持的服务和协议：telnet ftp pop3[-ntlm] imap[-ntlm] smb smbnt http-{head|get} http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3 mssql mysql oracle-listener postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn icq sapr3 ssh smtp-auth[-ntlm] pcanywhere teamspeak sip vmauthd firebird ncp afp等等。

OPT 可选项


1、破解ssh：

{% highlight %}
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh
hydra -l 用户名 -p 密码字典 -t 线程 -o save.log -vV ip ssh
{% endhighlight %}


2、破解ftp：

{% highlight %}
hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV
hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV
{% endhighlight %}

3、get方式提交，破解web登录：

{% highlight %}
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip http-get /admin/
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns -f ip http-get /admin/index.php
{% endhighlight %}

4、post方式提交，破解web登录：

{% highlight %}
hydra -l 用户名 -P 密码字典 -s 80 ip http-post-form "/admin/login.php:username=^USER^&password=^PASS^&submit=login:sorry password"
hydra -t 3 -l admin -P pass.txt -o out.txt -f 10.36.16.18 http-post-form "login.php:id=^USER^&passwd=^PASS^:<title>wrong username or password</title>"
{% endhighlight %}

（参数说明：-t同时线程数3，-l用户名是admin，字典pass.txt，保存为out.txt，-f 当破解了一个密码就停止， 10.36.16.18目标ip，http-post-form表示破解是采用http的post方式提交的表单密码破解,<title>中的内容是表示错误猜解的返回信息提示。）

5、破解https：

{% highlight %}
hydra -m /index.php -l muts -P pass.txt 10.36.16.18 https
{% endhighlight %}

6、破解teamspeak：

{% highlight %}
hydra -l 用户名 -P 密码字典 -s 端口号 -vV ip teamspeak
{% endhighlight %}

7、破解cisco：

{% highlight %}
hydra -P pass.txt 10.36.16.18 cisco
hydra -m cloud -P pass.txt 10.36.16.18 cisco-enable
{% endhighlight %}

8、破解smb：

{% highlight %}
hydra -l administrator -P pass.txt 10.36.16.18 smb
{% endhighlight %}

9、破解pop3：

{% highlight %}
hydra -l muts -P pass.txt my.pop3.mail pop3
{% endhighlight %}

10、破解rdp：

{% highlight %}
hydra ip rdp -l administrator -P pass.txt -V
{% endhighlight %}

11、破解http-proxy：

{% highlight %}
hydra -l admin -P pass.txt http-proxy://10.36.16.18
{% endhighlight %}

12、破解imap：

{% highlight %}
hydra -L user.txt -p secret 10.36.16.18 imap PLAIN
hydra -C defaults.txt -6 imap://[fe80::2c:31ff:fe12:ac11]:143/PLAIN
{% endhighlight %}

---

>PS
>1.hydra是开源软件，如果比较熟悉编译安装，可以自己下载源码安装，不过作为一个小白，我表示我不会。。。
>2.密码字典还是很重要的，暴力破解我个人认为主要还是破解弱口令有用，口令设复杂点想要暴力破解时间上也不允许。