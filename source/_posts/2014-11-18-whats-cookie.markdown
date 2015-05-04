---
layout: post
title: "what's cookie"
date: 2014-11-18 10:45:25 +0800
comments: true
categories: 
---

<!--more-->
###Cookie使用的典型场景
* 使用用户名和口令，通过 SSL 登录
* 在服务器的数据库里检查用户名和口令。如果登录成功，建立一个当前时间标签的消息摘要 (比如 MD5) ，并把它保存在cookie和服务器数据库里。把用户的登录时间保存在服务器数据库里面的用户记录里
* 在进行每个安全事务时（用户处于登录状态的任何事务），把cookie的消息摘要和保存在服务器数据库里的摘要进行比较，如果比较失败，就把用户引导到登录界面
* 如果第3步检查通过，那么检查当前时间和登录时间之音经过的时间是否超过允许的时间长度。如果用户已经超时，那么就把用户引到登录界面。
* 如果第3步和第4步都通过了，那么把登录时间重新设置成当前时间，允许事务发生。
###Cookie Sample
    name=<value>[; expires=<date>][; domain=<domain>][; path=<path>][; secure]

###Cookie的构成
在Javascript脚本里，一个cookie 实际就是一个字符串属性。当你读取cookie的值时，就得到一个字符串，里面当前WEB页使用的所有cookies的名称和值。每个cookie除了name名称和value值这两个属性以外，还有四个属性。这些属性是： expires过期时间、 path路径、 domain域、以及 secure安全。

* Expires – 过期时间。指定cookie的生命期。具体是值是过期日期。如果想让cookie的存在期限超过当前浏览器会话时间，就必须使用这个属性。当过了到期日期时，浏览器就可以删除cookie文件，没有任何影响。

* Path – 路径。指定与cookie关联的WEB页。值可以是一个目录，或者是一个路径。如果"http://www.zdnet.com/devhead/index.html" 建立了一个cookie，那么在"http://www.zdnet.com/devhead/" 目录里的所有页面，以及该目录下面任何子目录里的页面都可以访问这个cookie。这就是说，在"http://www.zdnet.com/devhead/stories/articles" 里的任何页面都可以访问"http://www.zdnet.com/devhead/index.html" 建立的cookie。但是，如果"http://www.zdnet.com/zdnn/" 需要访问"http://www.zdnet.com/devhead/index.html" 设置的cookies，该怎么办？这时，我们要把cookies的path属性设置成“/”。在指定路径的时候，凡是来自同一服务器，URL里有相同路径的所有WEB页面都可以共享cookies。现在看另一个例子：如果想让"http://www.zdnet.com/devhead/filters/" 和"http://www.zdnet.com/devhead/stories/"共享cookies，就要把path设成“/devhead”。

Domain – 域。指定关联的WEB服务器或域。值是域名，比如zdnet.com。这是对path路径属性的一个延伸。如果我们想让catalog.mycompany.com 能够访问shoppingcart.mycompany.com设置的cookies，该怎么办? 我们可以把domain属性设置成“mycompany.com”，并把path属性设置成“/”。FYI：不能把cookies域属性设置成与设置它的服务器的所在域不同的值。

Secure – 安全。指定cookie的值通过网络如何在用户和WEB服务器之间传递。这个属性的值或者是“secure”，或者为空。缺省情况下，该属性为空，也就是使用不安全的HTTP连接传递数据。如果一个 cookie 标记为secure，那么，它与WEB服务器之间就通过HTTPS或者其它安全协议传递数据。不过，设置了secure属性不代表其他人不能看到你机器本地保存的cookie。换句话说，把cookie设置为secure，只保证cookie与WEB服务器之间的数据传输过程加密，而保存在本地的cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。
