---
layout: post
title: "Peachfuzzer初探"
date: 2017-03-30 19:08:09 -0800
categories: 学习笔记
tags: PeachFuzzer Fuzzing Modbus TCP
---

之前一个项目使用Peachfuzzer发现了一款产品的漏洞，直接能造成该产品挂掉，身为一个小白，当然觉得很神奇，所以一直想学会怎么用这款fuzzing工具。

本文将教你怎么用Peachfuzzer对Modbus TCP server进行Fuzzing测试。

---

<h3>step1 下载安装Peachfuzzer</h3>
[PeachFuzzer官网]提供community editon的免费下载使用，从官网上看有V2和V3两种选择，小白嘛，从来只下最新的。

关于运行平台，推荐采用windows平台zip包，因为听说这玩意是用C#写的。

说是安装其实就是解压缩，根据官网的[安装教程]，需要安装：

1.Microsoft.NET v4 Runtime

2.Debugging Tools for Windows

但是好像2是一些高阶用法才用得到，所以小白就装1就好了。

---

<h3>step2 编写Peach Pit</h3>

就是.xml文件啦，Peachfuzzer会按照该文件运行。

关于这部分，我非常想吐槽一下[Peach官网教程]，写的太简略了，让我怀疑这个产品是不是老外做的，对比其他老外写的用户手册，很多地方都语焉不详，而且居然还会有没写完的地方...喂，好歹也好几年了吧！

![content-coming-soon.png](/images/peach-content-coming-soon.png)

不过作为小白，还是应该仔细学习一下教程，毕竟所有的配置都在这个.xml文件里。

下面我简要介绍一下我写的关于Modbus TCP协议的.xml文件：

**1.文件框架**

这部分对于Peach V3都一样的，如下：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<Peach xmlns="http://peachfuzzer.com/2012/Peach" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://peachfuzzer.com/2012/Peach ../peach.xsd">

<!-- your content -->

</Peach>
{% endhighlight %}

中间的主要部分包括DataModel、StateModel和Test。

**2.DataModel**

这部分用来定义发送数据的格式：

{% highlight xml %}
<DataModel name="Modbus">

	<!-- MBAP Header -->
	<Block name="MBAPHeader">
		<Number name="trans_id" value="1234" size="16" valueType="hex" endian="big" mutable="false" />	
		<Number name="proto_id" size="16"  value="0000" valueType="hex" endian="big" mutable="false" />
		<Number name="len" size="16" valueType="hex" endian="big" mutable="false">
			<Relation type="size" of="ChoiceFunc" expressionSet="size+1" />
		</Number>		
		<Number name="unit_id" size="8" value="01" valueType="hex" mutable="false" />		  
	</Block>
		
</DataModel>	
	
<DataModel name="Payload" ref="Modbus">
	<!-- PDU -->
	<Choice name="ChoiceFunc">
	<!-- Function code 1 -->	
		<Block name="Func1_readcoils">		
			<Number name="func_code" size="8" value="01" valueType="hex" endian="big" mutable="false" />
			<Number name="start_addr" size="16" value="0001" valueType="hex" endian="big" mutable="false" />
			<Number name="qty" size="16" value="0001" valueType="hex" endian="big" mutable="false" />		
		</Block>
			
	<!-- Function code 2 -->	
		<Block name="Func2_readdiscreteinputs">		
			<Number name="func_code" size="8" value="02" valueType="hex" endian="big" mutable="false" />
			<Number name="start_addr" size="16" value="0002" valueType="hex" endian="big" mutable="false" />
			<Number name="qty" size="16" value="0002" valueType="hex" endian="big" mutable="false" />		
		</Block>	
		
	</Choice>
</DataModel>
{% endhighlight %}

是完全按照ModbusTCP的协议格式写出来的，其中Payload是最终发出的所有应用层数据，使用了一个Choice发不同功能码的报文，由于篇幅关系，我就列出了功能码1和2的数据结构。

有参数看不懂的话，里面用到的所有参数在[Peach官网教程]都有较详细的解释~~，因为没有详细解释的我都不会用~~。

**3.StateModel和Test部分**

StateModel是规定了动作，说实话这部分不是特别懂，在本文里可以视为把数据发出去。

{% highlight xml %}
<StateModel name="ModbusStateModel" initialState="InitialState">
	<State name="InitialState">
		<Action type="output">
			<DataModel ref="Payload" />
		</Action> 
	</State>
</StateModel>
{% endhighlight %}

Test部分也不是很懂，好像就是调用了动作，然后指定了IP和端口。

{% highlight xml %}
<Test name="Default">
	<StateModel ref="ModbusStateModel" />
				
	<Logger class="Filesystem">
		<Param name="Path" value="Logs" />
	</Logger>

	<Publisher class="TcpClient">
		<Param name="Host" value="192.168.10.10"/>
		<Param name="Port" value="502"/>
	</Publisher>
</Test>
{% endhighlight %}

把这三部份拼起来（2、3放到1的中间），就组成了所需要的.xml文件。

<h3>step3 运行Peach</h3>

准备一个TCP的服务器开放502端口用于测试。

然后在Peach的目录下运行cmd命令：**Peach 文件名.xml**就行了。

现在可以通过抓包工具看看是否发送成功了。

---

>PS
>1.MBAP Header里有一个参数是后续报文的字节数，这块我想了半天也不知道怎么实现，发出去的fuzzing报文的总长度变化时这个数字总是只考虑DataModel里的长度，后来我想了想，可能这个该工具没打算让你把额外的畸形报文长度考虑进去。
>
>2.关于fuzzing我一开始以为是在规定的数据结构里对所有的参数进行遍历，但我把DataModel所有的element全部设置为mutable=false的时候依然可以发出来畸形报文，我才发现所谓的Fuzzing是对整个报文的结构和数据都会去进行测试，所以Fuzzing的算法看来是一门很深奥的学问。
>
>3.[Peach官网教程]如果解决不了问题，可以去[Peach官方论坛]找找灵感。

就这么多吧。

[PeachFuzzer官网]: http://www.peachfuzzer.com
[安装教程]: http://community.peachfuzzer.com/v3/Installation.html
[Peach官网教程]: http://community.peachfuzzer.com/v3/PeachPit.html
[Peach官方论坛]: https://forums.peachfuzzer.com/