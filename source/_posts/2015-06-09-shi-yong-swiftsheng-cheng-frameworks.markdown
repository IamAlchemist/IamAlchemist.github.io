---
layout: post
title: "使用swift生成Frameworks"
date: 2015-06-09 19:04:02 +0800
comments: true
categories: iOS
---

###制作Dynamically Linked Frameworks  
<!--more-->

###一个普通的Swift file  
	class SwiftFrameworks {
		init() { println("Class init") }
		func doSomething() { println("Yeah, it works") }
	} 

为了让它可以在framework中工作，需要让所有类和方法都是*public*的

	public class SwiftFrameworks { 
		public init() { println("Class init") } 
		public func doSomething() { println("Yeah, it works") } 
	} 


###建立framework工程  
{% img center /images/iOS/2015-06-09_1.png 600 %}

###添加刚才的swift文件
{% img center /images/iOS/2015-06-09_4.png 600 %}

###编译
有时候编译时候并不能产生target，切换一下设备（可能是xcode的bug），然后找到build好的target所在文件夹

{% img center /images/iOS/2015-06-09_2.png 600 %}

###添加到测试工程中
把编译好的framework拖拽到测试工程中，之后在embed中添加该framework

{% img center /images/iOS/2015-06-09_3.png 600 %}

然后

	import SwiftFramework
	let swiftFramework = SwiftFramework()
	swiftFramework.helloWorld()

大功告成～








