---
layout: post
title: "小白如何搭建个人博客"
date: 2017-03-27 17:08:09 -0700
categories: 学习笔记
tags: Jekyll Renge Github Wordpress
---

最近一直想搭建一个博客，了解了一些相关知识后，准备采取：

**Lamp+Wordpress+DDNS**的方式。

这三样东西也很快就准备好了，可是装好以后才发现有不少的问题：

1.Wordpress虽好，但是略显复杂和臃肿

2.免费的DDNS简直不能用，例如~~花生壳~~，壳域名直接被某讯标记为危险网站，而且域名解析时灵时不灵

3.以后网站迁移比较麻烦，身为菜鸟中的菜鸟，数据库啥的都不知道怎么迁移TT

4.逼格好像还是低了点，既然要装逼就要装得更像

所以经过了解，决定采用Jekyll将博客发布在Github，以下就教你整个过程，就算是小白也能搭建一个自己的博客。

---
<h3>step1 注册Github</h3>
这步我想应该不用教了吧。

---
<h3>step2 创建新的Repository</h3>
注意名称必须符合格式要求，否则之后的步骤中项目页面的路径会多一些东西，对小白而言简直是灾难。

![creat-new-repository](/images/creat-new-repository.png)

其余的全部默认不用改。

---
<h3>step3 下载Github客户端</h3>
下载链接： [Github Desktop]

这步可能会遇到阻碍，因为这个网址虽然能打开，但是下载的话貌似很慢很慢，据说与~~GFW~~有关，反正不管是翻墙还是从别的地方下载安装包，这个随意啦。

---
<h3>step4 下载Jekyll模版</h3>
身为一个小白，当然是下载别人好看又炫酷的模版啦

作为本站的例子，我使用了Junchao的Jekyll模版<a href="https://github.com/billyfish152/Renge">Renge</a>，喜欢可以下载使用，MIT的license。

---
<h3>step5 上传Jekyll模版至Github</h3>
首先登录Github客户端，将新建的Repository克隆到本地。

![clone-repository-to-local](/images/clone-repository-to-local.png)

然后把Jekyll模版copy到本地的Repository的目录。
Github客户端就会发现Repository出现了更改，对更改做出确认后发布即可。

---
<h3>step6 完成测试</h3>
用浏览器访问自己的页面试试吧，是不是很简单！

---
<h3>PS</h3>
>1.一开始我以为Github page只是类似于apache的web服务器，把Jekyll编译之后的静态页面发布了上去，后来才发现too young。
>
>2.关于Jekyll，可以自己在本地装一个然后好好研究，这样就没必要每次都要发布到Github。
>
>3.关于安装Jekyll，我只能说网上的教程很坑，都是从安装ruby开始教起，其实Ubuntu下只要运行apt-get install jekyll就好了。

[Github Desktop]: https://desktop.github.com/