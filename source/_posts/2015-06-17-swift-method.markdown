---
layout: post
title: "Swift Method"
date: 2015-06-17 12:42:28 +0800
comments: true
categories: iOS swift
---

swift `method`分为`instance method`和`type method`

`instance method`是属于实例的函数，`type mothod`则是属于`type`的函数

<!--more-->
### Local and External Parameter Names for Method
`local and external parameter name`的默认行为并不等同于`function`

swift的method的参数名字非常像objective－c。比如说，一般来说，swift的method的名字带有by， with， for等介词，例如incrementBy。

需要指出的是，swift默认会给第一个参数local parameter name，但是对于第二个以后参数则默认生成local and external parameter name。
	class Counter {
		var count : Int = 0
		func incrementBy(amount: Int, numberOfTimes: Int){
			count += amount * numberOfTimes
		}
	}
swift会把amount看作local name，但是会把numberOfTimes看作local and external name。所以调用该method，需要像如下这样
	let counter = Counter()
	counter.incrementBy(5, numberOfTimes: 3)
当然也可以显式的提供第一个参数的external name或者不提供非第一参数的external name
	class Counter2{
		var count : Int = 0
		func incrementBy(#amount: Int, _ numberOfTimes: Int){
			count += amount * numberOfTimes
		}
	}
	let count2 = Count2()
	count2.incrementBy(amount: 5, 3)

### Self Propety
一般情况下可以不写，但是如果有歧义就需要写
	struct Point{
		var x = 0.0, y = 0.0
		func isToTheRightOfX(x: Double) -> Bool{
			return self.x > x   // must use self
		}
		func description() -> String{
			return "x: \(x), y: \(y)") //no need to use self
		}
	}

### 在instance method内改变值类型变量自身
对于`value types`来说，一般情况下，instance method不能更改属性，但是加上mutating关键字来改变这点。
	struct Point{
		var x = 0.0, y = 0.0
		mutating func moveByX(deltaX: Double, y deltaY: Double){
			x += deltaX
			y += deltaY
		}
	}

### Assigning to self Within a Mutating Method
	struct Point{
		var x = 0.0, y = 0.0
		mutating func moveByX(deltaX: Double, y deltaY: Double){
			self ＝ Point(x: x+deltaX, y: y + deltaY)
		}
	}

	enum TriStateSwitch {
		case Off, Low, High
		mutating func next() {
			switch self {
			case Off:
				self = Low
			case Low:
				self = High
			case High:
				self = Off
			}
		}
	}

### Type Method
`type method`在类中使用关键字`class`来表示，在结构体和枚举中使用`static`来表示

	
