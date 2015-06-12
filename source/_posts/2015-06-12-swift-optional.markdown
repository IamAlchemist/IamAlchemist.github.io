---
layout: post
title: "swift中optional"
date: 2015-06-12 15:37:14 +0800
comments: true
categories: iOS swift
---

关于optional，最需要说的是为什么需要optional，比如说在objctive-c中就没有，也是可以表示一个不存在的对象的。只需要用NULL就可以了。但是如何表示一个不存在的value类型的变量呢？比如一个不存在的struture？objective-c就没有太好的办法了，一般来说是用一个特殊值NSNotFound来表示。不过swift可以让一切不存在的变量都为nil，这就是optional的目的

<!--more-->

关于optional，剩下的只是swift里面有关的特殊术语，搞清楚还是很有必要的

####Force Unwrapping

Force Unwarpping指的是把optional对象转化成一个非空的对象

####Optional Binding

其基本形式是

	if let constantName = someOptional

	while let constantName = someOptional
	
####Implicity Unwrapped Optionals

一个optinal变量可以定义为`implicity unwrapped optionals`, 那么使用这个变量的时候就不再需要force unwrap, 一旦unwrap失败，程序会崩溃

	var storyboardName : String! = "hello"


	

	
