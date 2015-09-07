
---
layout: post
title: "到底能不能在extension里override一个函数？"
date: 2015-07-22 11:12:03 +0800
comments: true
categories: iOS Swift
---

Swift教程中明确说了，extension并不能override一个已有的函数！可是最近发现有人extension UIImageView时，可以`override layoutSubviews()`，到底什么鬼？？

<!--more-->

在[stackoverflow](http://stackoverflow.com/questions/27109006/can-you-override-between-extensions-in-swift-or-not-compiler-seems-confused)上找到了答案，并且自己也确认过了。

解释是这样的：

至少在目前版本（swift1.1， 1.2），只要在如下两种情况下就可以override函数

* 涉及到的类都是从NSObject继承来的,不使用inout修饰符并且没有enum
* 或者使用了@objc修饰符的函数

核心思想是，只有*Objective-C compatible*的方法和属性才能在extension里override

请参考下面的例子

####例子1

{% codeblock lang:swift %}

class A : NSObject { }
class B : A { }

class SubNSObject : NSObject {}
class NotSubbed {}
enum SomeEnum { case c1, c2; }

extension A
{
    var y : String { get { return "YinA"; } }
    func f() -> A { return A(); }
    func g(val: SubNSObject, test: Bool = false) { }

    func h(val: NotSubbed, test: Bool = false) { }
    func j(val: SomeEnum) { }
    func k(val: SubNSObject, inout test: Bool) { }
}

extension B 
{
    // THESE OVERIDES DO COMPILE:
    override var  y : String { get { return "YinB"; } }
    override func f() -> A { return A(); }
    override func g(val: SubNSObject, test: Bool) { }

    // THESE OVERIDES DO NOT COMPILE:
    //override func h(val: NotSubbed, test: Bool = false) { }
    //override func j(val: SomeEnum) { }
    //override func k(val: SubNSObject, inout test: Bool) { }
}

{% endcodeblock %}

####例子2

{% codeblock lang:swift %}

class A { }

class B : A { }

extension A
{
    @objc var y : String { get { return "YinA" } }
}

extension B
{
   @objc override var y : String { get { return "YinB" } }
}

{% endcodeblock %}
