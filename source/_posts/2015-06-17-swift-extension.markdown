---
layout: post
title: "swift extension"
date: 2015-06-17 07:45:07 +0800
comments: true
categories: iOS swift
---

`extensions`可以给已有的type增加功能，而且不必访问源代码(`retroactive modeling`)

`extensions`可以
* 增加`computed property`和`computed static property`
* 定义`instance methods`和`type methods`
* 提供新的`initializers`
* 定义`subscripts`
* 定义和使用新的`nested types`
* 为一个`type`实现一个`protocol`

`extension`不可以
* `extension`可以增加新的`funtionality`，但是不能覆盖以有的`functionality`
* `extension`不能增加`stored properties`，或者给存在的`property`增加`property observers`
* `extension`不能增加新的`designated initializer`

<!--more-->

#### Extension 语法
	extension SomeType {
		...
	}

	extiosion SomeType: Someprotocol, AnotherProtocol {
		...
	}

一个有趣的例子

	extension Double {
		var km: Double { return self * 1000.0 }
		var m: Double { return self }
	}

	let aMarathon = 42.m + 195.m
	println("A marathon is \(aMarathon) masters long")

#### Initializers
	struct Size { var width = 0.0, height = 0.0 }
	struct Point { var x = 0.0, y = 0.0 }
	struct Rect {
		var origin = Point()
		var size = Size()
	}

	extension Rect{
		init(center: Point, size: Size){
			let originX = center.x - (size.width / 2)
			let originY = center.y - (size.height / 2)
			self.init(origin: Point(x:originX, y:originY), size: size)
		}
	}

#### Methods
	extension Int{
		func repetitions(task: ()-> ()){
			for i in 0..<self {
				task()
			}
		}
	}

	3.repetitions{ println("hello") }

####Mutating Instance Methods
	extension Int {
		mutating func square(){
			self = self * self
		}
	}

	var someInt = 3.square()

#### Subscripts
	extension Int {
		subscript(var digitIndex: Int) -> Int{
			var decimalBase = 1
			while digitIndex > 0 {
				decimalBase *= 10
				--digitIndex
			}

	        return (self / decimalBase) % 10
		}
	}

	74631295[1] //return 9

#### Nested Tyeps
	extension Int {
		enum Kind {
			case Negative, Zero, Positive
		}

	    var kind: Kind {
			switch self {
			case 0: retrun .Zero
			case let x where x > 0: return .Positive
			default: return .Negative
			}
		}
	}
	
