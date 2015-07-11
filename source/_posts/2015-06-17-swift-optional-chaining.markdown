---
layout: post
title: "Swift Optional Chaining"
date: 2015-06-17 09:19:59 +0800
comments: true
categories: iOS swift
---

`optinal chainging`是一个查询和调用`optional`的property, method, subscript的过程。如果其中的`optional`没有nil，那么表达式求值成功。否则，失败。失败后返回nil。所以整个表达式返回值始终是`optional`类型。

<!--more-->

以下是一些sample code, 先构造一些类

	class Person{ var residence: Residence? }

	class Residence {
		var rooms = [Room]()
		var numberOfRooms: Int { return rooms.count }

		subscript(i: Int) -> Room {
			get { return rooms[i] }
			set { rooms[i] = newValue }
		}
		
		func printNumberOfRooms(){
			println("The number of rooms is \(numberOfRooms)")
		}
	}

	class Room {
		let name: String
		init(name: String){ self.name = name }
	}

	class Address {
		var buildingName: String?
		var buildingNumber: String?
		var street: String?
		
		func buildingIdentifier() -> String? {
			if buildingName != nil {
				return buildingName
			} else if buildNumber != nil {
				return buldingNumber
			} else {
				return nil
			}
		}
	}

下面就是如何使用`optional chain`

	let john = Person()
	if let roomCount = john.residence?.numberOfRooms {
		println("john's residence has \(roomCount) rooms.")
	}
	// will print nothing

	let someAddress = Address()
	someAddress.buildingNumber = "29"
	someAddress.street = "Acacia Road"
	john.residence?.address = someAddress //will fail

Residence里的方法printNumberOfRooms()没有返回值，但是这意味着该方法返回类型是Void，也就是返回值是(),也就是一个空的tuple. 在`optional chain`里，会返回Void?

	if john.residence?.printNumberOfRoom() != nil {
		println("It was possible to print the number of rooms."
	} 
	// will print nothing

对于通过`optional chain`来设置属性也一样

	if (john.residence?.address = someAddress) != nil {
		println("It was possible to set the address."
	}
	// will print nothing

通过`optional chaining`访问subscript

	if let firstRoomName = john.residence?[0].name {
		println("the first room name is \(firstRoomName)")
	}
	// will print nothing

	let johnsHouse = Residence()
	johnsHouse.rooms.append(Room(name: "Living Room"))
	johnsHouse.rooms.append(Room(name: "Kitchen"))
	john.residence = johnsHouse

	if let firstRoomName = john.residence?[0].name {
		println("the first room name is \(firstRoomName)")
	}
	// prints "the first room name is Living Room"

访问`optional`的subscript

	var testScrores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
	testScores["Dave"]?[0] = 91
	testScores["Bev"]?[0]++
	testScores["Brain"]?[0]

多层的optional chain， 原则是
* 如果试图查询的东西的类型并非是`optional`的，会由于使用了chain而变成`optional`
* 如果正在查询的东西已经是`optional`的，它不会因为chain变得“更”`optional`（`optional`的`optional`)

下面是代码示例

	if let johnsStreet = john.residence?.address?.street{
		println("John's street name is \(johnsStreet)")
	}
	// print nothing

	let johnsAddress = Address()
	johnsAddress.buildingName = "The Larches"
	johnsAddress.street = "Laurel Street"
	john.residence?.address = johnsAddress

	if let johnsStreet = john.residence?.address?.street{
		println("John's street name is \(johnsStreet)")
	}
	// print "John's street name is Laurel Street"

	if let beginsWithThe = john.residence?.address?.buildingIdentifier()?.hasPrefix("The") {
		if beginWithThe {
			println("John's building identifier begins with \"The\".")
		}
	}
