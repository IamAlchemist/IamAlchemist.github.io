---
layout: post
title: "Swift中的类型转换"
date: 2015-06-11 11:34:01 +0800
comments: true
categories: iOS Swift
---

Swift中使用`as`和`is`运算符来进行类型转换

<!--more-->
####一个类继承的例子

	class MediaItem{
	}

	class Movie: MediaItem{
		var director : String
		init(director: String){ self.director = director; super.init() }

	}

	class Song: MediaItem{
		var artist : String
		init(artist: String){ self.artist = artist; super.init() }
	}

####类型检查使用`is`

	let library = [Movie("d1"), Song("a1"), Song("a2"), Movie("d2")]

	var movieCount = 0
	var songCount = 0

	for item in library {
		if item as Movie {
			++movieCount
		} else if item is Song {
			++songCount
		}
	}

值得一提的是，`is`不仅仅用来检测有继承关系的类，而且可以用来检测是否实现了某个接口

	protocal HasArea{
		var area: Double { get }
	}

	class Circle: HasArea{
		var radius: Double
		var area: Double { return 3.14 * radius * radius }
		init(radius: Double){ self.radius = radius }
	}

	var circle = Circle(2.5)
	if circle is HasArea {
		println("Area: \(circle.area)")
	}

####类型转换

可以使用`as`来进行类型转换

可以用两种方式来使用`as`, `as?`和`as!`

`as?`表示不确定该转换是否能成功，`as!`表示一定会成功，但是如果运行时转换失败的话，程序会崩掉

	for item in library {
		if let movie = item as? Movie {
			println("Director: \(movie.director)")
		} else if let song = item as? Song {
			println("Artist: \(song.artist)")
		}
	}

####`Any`和`AnyObject`的类型转换

其实，`Any`和`AnyObject`的转换和普通的类型转换是一样的。值得说的是为什么swift要有他们两个。

* `AnyObject`可以代表任何class类型的实例
* `Any`却可以代表任何类型的实例，包括funtion类型

举两个例子，第一个例子

	let someObjects: [AnyObject] = [Movie('d1'),Movie('d1'),Movie('d1')]
	for object in someObjects{
		let movie = object as! Movie
		println("Movie director: \(movie.director)")
	}

该例子中，还可以直接cast整个数组

	for movie in someObjects as! [Movie] {
		println("Movie director: \(movie.director)")
	}

另外一个是关于`Any`的例子

	var things = [Any]()
	
	things.append(0)
	things.append(0.0)
	things.append(42)
	things.append(3.14)
	things.append("hello")
	things.append((3.0, 5.0))
	things.append(Movie("d1"))
	things.append({(name: String) -> String in 
		"Hello, \(name)" })

	for thing in things{
		switch thing{
			case 0 as Int: 
				println("zero as an Int")
			case 0 as Double:
				println("zero as an Double")
			case let someInt as Int:
				println("an integer value of \(someInt)")
			case let someDouble as Double where someDouble > 0:
				println("a positive double : \(someDouble)")
			case is Double:
				println("some other double value <= 0")
			case let someString as String:
				println("a string value : \(someString)")
			case let (x, y) as (Double, Double) :
				println("an (x,y) point at \(x), \(y)")
			case let movie as Movie:
				println("a movie director: \(movie.director)")
			case let stringConverter as String -> String:
				println(stringConverter("Michael")
			default:
				println("something else")
		}
	}

需要指出，在switch的case语句中，使用的`as`是强制转换版的，并不写作`as?`或者`as!`，在case语句中使用这种检查总是安全的。
	


	

