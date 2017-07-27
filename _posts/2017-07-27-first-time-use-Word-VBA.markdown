---
layout: post
title: "关闭Windows不安全的端口"
date: 2017-07-27 11:08:09 -0700
categories: 学习笔记
tags: Word VBA Office
---

最近遇到了很坑的任务，要精简一个包含大量各种表格的文档，几千页的东西要是手弄的话估计得跪，而且还**没有任何收获**。

所以终于静下心来学习一下Word VBA怎么用

---
<h3>1.Hello World</h3>
默认word是没有把VBA的功能放到面儿上的，需要在开启“**选项-自定义功能区-开发工具**”

![active-develop-tool](/images/active-develop-tool.png)

然后在开发工具面板打开“Visual Basic”，

![open-visual-basic](/images/open-visual-basic.png)

打开之后，插入模块，这就是Word VBA的编辑器了,

![VBA-editor](/images/VBA-editor.png)

在编辑器中写入以下的代码

{% highlight vba %}

Sub helloworld()
MsgBox "hello world!"
End Sub

{% endhighlight %}

运行这段代码，就会弹出对话框，

![VBA-hello-world](/images/VBA-hello-world.png)

是不是很容易，这就是VBA的基本使用方式了，除了插入模块，还可以插入用户窗体，不过这个就留待后面在研究吧。

---

<h3>2.让代码与文档交互</h3>

那么代码要怎么样才能对文档产生作用呢？这时候就必须了解VBA的**对象**，文档中的所有东西都是VBA的对象，**对象**具有相关的**属性**，**属性**能够赋予**值**，操作**对象**需要用到对应的**方法**

反正我一句两句话确实说不清（汗）

简单的例子：

在一个包含大量几种表格的文档里，删除特定类型表格的特定行。
例如下面，删除所有6列表格的包含“低”的行（预先知道“低”所处的列）

![example-table-1](/images/example-table-1.png)

{% highlight vba %}

Sub delete()
Dim col As Integer
col = 2 ' 要匹配的列

For i = 1 To ActiveDocument.Tables.Count
    If ActiveDocument.Tables(i).Columns.Count = 6 Then
        For j = 1 To ActiveDocument.Tables(i).Rows.Count
            If InStr(ActiveDocument.Tables(i).Cell(j, col).Range.Text, "低") <> Empty Then
            ActiveDocument.Tables(i).Rows(j).delete
            End If
        Next
    End If
Next

MsgBox "运行结束"
End Sub

{% endhighlight %}

运行结束后，就会发现指定的表格中包含“低”的行被删除了。

![example-table-1-after](/images/example-table-1-after.png)

---

<h3>3.目前学到的其他东西</h3>

1）删除行时有时会提示“纵向有合并的单元格”

使用选择方法可以解决这个问题

{% highlight vba %}

ActiveDocument.Tables(i).Cell(i, j).Select
Selection.Rows.delete

{% endhighlight %}

2）停止页面刷新，加快脚本运行速度

有的时候脚本运行时间非常长，加了下面语句后明显快了不少，记得在运行完开启页面刷新。

{% highlight vba %}

Application.ScreenUpdating = False

{% endhighlight %}

3）计算程序运行时间

Timer是系统当前时间，下面语句可以计算程序运行时间

{% highlight vba %}

time_start = Timer
time_end = Timer
MsgBox "OK，用时： " & time_start - time_end

{% endhighlight %}

4）复制一个表格到文章末尾（先输入一个回车）

{% highlight vba %}

Sub test()

ActiveDocument.Tables(1).Select
Selection.Copy

ActiveDocument.Range(Start:=ActiveDocument.Content.End - 1, End:=ActiveDocument.Content.End - 1).InsertAfter Chr(10)
ActiveDocument.Range(Start:=ActiveDocument.Content.End - 1, End:=ActiveDocument.Content.End - 1).Paste

MsgBox "ok"
End Sub

{% endhighlight %}

5）动态调整数组长度

VBA的数组一旦赋值就是定长的，想要调整数组长度就要用到ReDim重复声明变量，使用Preserve能够保留Redim之前的数组数据，例如如下代码将一个长度为3的数组调整至长度为5.

{% highlight vba %}

Sub test()
Dim a
a = Array("a", "b", "c")
ReDim Preserve a(5)
a(3) = "d"
a(4) = "e"
MsgBox a(0) & a(1) & a(2) & a(3) & a(4) & a(5)
End Sub

{% endhighlight %}

运行之后就能看到消息框输出“abcde”

6）有没有判断一个元素是否出现在数组的函数？

在Word VBA中反正我是没找到，据说在Excle VBA中有，所以判断一个元素在不在数组中还是需要自己写的。

---
大概就以上这些吧，有了上面这些，在对常见的判断、循环有所熟悉后，反正也能完成一些复杂的动作了。