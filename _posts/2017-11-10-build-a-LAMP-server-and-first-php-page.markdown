---
layout: post
title: "简单搭建一个基于LAMP的带鉴别的网页"
date: 2017-11-10 16:08:09 -0700
categories: 学习笔记
tags: Ubuntu PHP MySQL
---

想要自己搭点Web应用，但是由于不懂的东西实在是太多了，想要利用现成的框架或者模版都不知道从何下手。

所以先从造轮子开始吧。

---
<h3>1.安装集成了LAMP的Ubuntu</h3>
Ubuntu实在是个好东西，Ubuntu16.04.2在安装的时候直接可以选择LAMP服务器，简直是小白救星，否则后面自己逐个安装的话配置文件啥的能把人搞晕

![select-lamp-server](/images/select-lamp-server.png)

另外有需要的话也可以把邮件服务、smb和**SSH（为了方便最好选上）**都选上。

之后会提示配置MySQL root用户的口令，然后等着装完就行了。

<h3>2.配置工作</h3>

<h5>2.1 操作系统配置</h5>

一开始root用户密码是随机的，需要先配root用户密码

![set-ubuntu-root-password](/images/set-ubuntu-root-password.png)

配置网络（这个不会就上网查吧）

<h5>2.2 MySQL配置</h5>

用数据库管理工具一开始是无法远程连接MySQL的

![mysql-connect-error](/images/mysql-connect-error.png)

主要是要修改mysql配置文件和mysql数据库中user表（具体操作上网查吧）

<h5>2.3 SSH配置</h5>

配这个主要是为了方便用WinSCP传文件

一开始root用户是不能通过SSH远程登录的，需要修改SSH的配置文件。（这个不会就上网查吧）

<h3>3.使用PHP连接MySQL</h3>

~~当时我走了些弯路，在一个叫[W3school]的网站上说是这么连接的：~~

{% highlight php %}

mysql_connect(servername,username,password);

{% endhighlight %}

后来才知道，PHP版本升级了，使用**MySQLi extension**进行连接

使用WinSCP连接服务器，把/var/www/html目录下的index.html替换成下面这段代码，文件名变成index.php。

记得修改代码里的用户名和口令。

{% highlight php %}

<?php
$servername = "localhost";
$username = "username";
$password = "password";
 
// 创建连接
$conn = new mysqli($servername, $username, $password);
 
// 检测连接
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
} 
echo "连接成功";
?>

{% endhighlight %}

然后用浏览器访问一下你的服务器，应该就能看到连接成功了。

![connect-mysql-successed](/images/connect-mysql-successed.png)

<h3>4.做一个简单的带鉴别的页面</h3>

<h5>4.1 添加用户数据库</h5>

生成一个用户列表的数据库（可以手动添加，也可以用MySQLi生成，具体操作上网查吧）

![test-user-table](/images/test-user-table.png)

<h5>4.2 利用session实现登录</h5>

~~http是不带状态的协议，没办法判断用户是否登录，一般来讲主要是用cookie和session来实现的，简单来讲，cookie就是记录在浏览器端的状态，session是记录在服务端的状态，我目前知道的就是这么多了。~~

php自带了session的功能，我们直接拿来使用就好了。

我做了两个页面，一个登录页面index.php和一个登录成功的页面welcome.php

**index.php**

{% highlight php %}

<?php
$servername = "localhost";
$username = "root";
$password = "123qwe";
$dbname = "test";

session_start();//session需要先启动

//判断uname和upassword是否赋值 
if(isset($_POST['name'])&& isset($_POST['password'])){
	$uname = $_POST['name'];
	$upassword = $_POST['password'];

	//连接数据库
	$conn = new mysqli($servername, $username, $password, $dbname);
	if ($conn->connect_error) {
		die("连接失败: " . $conn->connect_error);
	} 

	//验证内容是否与数据库的记录吻合。
	$sql = "SELECT * FROM user WHERE name='$uname' and password='$upassword'";
 
	//执行上面的sql语句并将结果集赋给result。
	$result = $conn->query($sql);
 
	//判断结果集的记录数是否大于0
	if ($result->num_rows > 0) {
		$_SESSION['user_account'] = $uname;//session赋值
   
		// 若成功跳转页面，若失败则提示
		while($row = $result->fetch_assoc()) {
			header('Location: http://192.168.20.236/welcome.php');  
		}
	} 
	else {
		echo "用户名或密码错误";
	}
	$conn->close(); 
}
?>

<!DOCTYPE html>
<head>
<meta charset="UTF-8">
<title>登录</title>
</head>
<body>
<p>输入用户名和口令</p>
<form method="POST">
<input type="text" name="name" placeholder="用户名" />
<br />
<input type="text" name="password" placeholder="密码" />
<br />
<input type="submit">
</form>
</body>
</html>

{% endhighlight %}

访问是这种效果：

![my-first-login-page](/images/my-first-login-page.png)

**welcome.php**

{% highlight php %}

 <?php
session_start();
//检查session，判断是否登录
if(isset($_SESSION['user_account'])){
	echo '登录成功！';
}
//未登录跳转回登录页面
else{
	header('Location: http://192.168.20.236');
}
 ?>

{% endhighlight %}

使用test1/123qwe登录后：

![my-first-login-success](/images/my-first-login-success.png)

---
<h3>PS</h3>

以上只是实现了最简单的登录功能，很多安全方面的问题都欠考虑，至少对输入的字符没有做任何过滤，而且页面跳转的方式大概也有问题，可能实际并不是这么操作的。~~不过毕竟是造轮子，大概知道原理就好了，后面还是得依靠现成的框架才是正路。~~

[W3school]：http://www.w3school.com.cn/php/php_mysql_connect.asp