---
layout: post
title: "swift学习笔记 -- 函数与闭包"
description: "swift学习笔记"
category: "2015-07"
tags: [swift,closure]
---


每一门语言都离不开函数这个东西，swift当然就不会例外了。

swift的函数命名方式跟Objective-C可以说是一脉相承，非常好的表述了函数实现的功能。swift的语法也是它能够像表述像c类型这种没有参数名字，也能表述像Objectivie-C一样的有内部和外部名字的参数形式。也能为函数的参数提供默认值和引用参数(in-out)这样的方式去调用。

在swift中，function是 first class ，说明它是跟其它的值类型没有什么区别的，可以赋值，作为函数参数和作为返回值，这个特性就是使 swift 作为一个 functional programming language 成为可能。


### 定义和调用函数

定义时，可以选择定义一个或者多个参数作为输入，以及为参数定一个名字。调用时，用名字以及可能对应的参数顺序。函数是以一个 `func` 作为关键词开始。

	func sayHelloWorld(personName : String) -> String {
		let greeting = "Hello world , " + personName + "!"
		return greeting
	}

	sayHelloWorld("Niko")  
	//output  : Hello world , Niko!

通过使用 `->` 标明它的返回值。

### 参数

每一个参数都有一个名字，用于描述参数的作用，这个名字又分为外部使用和内部使用的。

	func generateFullName(fistName firstName: String , familyName : String) -> String {
		return firstName + " " + familyName
	}

	generateFullName(fistName: "Niko", familyName: "Tung")
	// Niko Tung


第一个参数的外部名称 `familyName` 默认是忽略的,所以一般的的写法是这样的
	
	func generateFullName(firstName: String , familyName : String) -> String {
		....
	}

	generateFullName("Niko", familyName: "Tung")

如果想让其它参数的名称在函数调用的时候也忽略，可以这样写：

	func generateFullName(firstName: String , _ familyName : String) -> String {
		....
	}

	generateFullName("Niko", "Tung")


也可以为参数指定默认值

	func generateFullName(firstName: String , _ familyName : String = "unknow") -> String {
		....
	}

	generateFullName("Niko") //Niko unknow
	generateFullName("Niko" , "Tung") //Niko Tung

在swift的函数中，参数默认都是常量的，这就意味着你不能在函数体内修改参数的值。所以如果需要在函数体内修改参数的值，你需要把它定义成变量。就是在参数名字前面加上	`var`去声明就行了。

	func generateFullName(var  firstName: String , _ familyName : String = "unknow") -> String {
		....
	}

### 函数类型(Function Type)

在我们的 oop 编程中，一般说函数的类型都是指函数的返回类型。在 swift 中是不一样的。函数在 swift 中是一等公民(`first-class citizens`),跟其它基本类型值是一样的。

	func addTwoInts(a : Int , _ b : Int) -> Int {
		return a + b
	}

	func multiplyTowInts(a : Int , _ b : Int) -> Int {
		return a * b
	}

上面的两个方法的函数类型可以这样描述：(Int , Int) -> Int 函数接收两个整形参数，最后返回一个整形结果。

这样的函数类型，可以像一般的值一样，通过赋值可以获取定义,并且只要函数的类型一样，它可以赋值给同样的变量

	var mathFunction: (Int , Int) -> Int = addTwoInts

	print("result : \(mathFunction(2,3))")
	//print "result : 5"

	mathFunction = multiplyTowInts

也可以把函数类型作为一个方法的参数

	func printMathResult(mathFunction : (Int , Int) -> Int , _ a : Int , _ b : Int) {
		print("Resutl : \(mathFunction(a , b))")
	}

	printMathResult(addTwoInts , 3, 5)
	//print "Result : 8" 

当然也可以把函数类型当让一个函数的返回值

	func getTheAddTwoIntsFucntion() -> (Int , Int) -> Int {
		return addTwoInts
	}

也可以写成这样
	
	func getTheAddTwoIntsFucntion() -> (Int , Int) -> Int {
		func addTwoInts(a : Int , _ b : Int) -> Int {
			return a + b
		}

		return addTwoInts
	}

### 闭包(Closures)

闭包其实就像一个函数一样，一段可以执行的代码，有输入输出，能够在代码中传递像函数的参数一样。

全局函数和嵌套函数(Global and nested function),其实就是闭包的一种。

得益于 swift 的编译器的强大，闭包的语法可以变得很简洁。

* 从上下文可以自动推断参数和返回值(Inferring parameter and return value types from context)
* 在简单表达式中隐式返回(Implicit Returns from Single-Expression Closures)
* 参数名称缩写(Shorthand argument names)
* 尾随闭包(Trailing closure syntax)

### 闭包表达式语法

	{(parameters) -> return type in 
		statements
	}

这里其实很像前面的函数的声明一样，用关键词 `in` 隔开闭包的类型声明和闭包体。

看看排序的方法

	//     func sort(isOrderedBefore: (Self.Generator.Element, Self.Generator.Element) -> Bool) -> [Self.Generator.Element]

	let names = ["Chris" , "Alex" , "Ewa" , "Barry" , "Daniella"]

	func backwards(s1 : String , s2 : String) -> Bool {
		return s1 > s2
	}

	var reversed = names.sort(backwards)
	//reversed now is ["Ewa", "Daniella", "Chris", "Barry", "Alex"]

这里的`backwards`方法比较两个字符串的大小，如果`s1`大于`s2`则返回 true 。而在sort方法里面的参数是`isOrderedBefore`，说明`backwards`中返回 true 了就排在前面。

用闭包的方法可以这样写

	reversed = names.sort({ (s1 : String , s2 : String) -> Bool in 
			return s1 > s2
		})

这就像 Go 语言里面的的匿名函数了。

### 类型推断

因为在上面的 sort 函数的闭包中，它是以一个参数的形式传递给 sort 函数的。swift 就可以通过 sort 这个函数就能通过它自己的参数自动推断出传递过来的参数的类型和返回值了,`(String , String) -> Bool`。

一旦 swift 能够自动推断出的事情，我们就不需要去重复做了，所以排序的闭包可以简化成这样

	reversed = names.sort({ s1 , s2 in return s1 > s2})

> It is always possible to infer the parameter types and return type when passing a closure to a function as an inline closure expression. As a result, you never need to write an inline closure in its fullest form when the closure is used as a function argument. 

### 在简单表达式中隐式返回

上面的排序我们可以再进一步简化成这样：不需要写 `return`

	reversed = names.sort({s1 , s2 in s1 > s2})

这也是因为`sort`函数中的参数中能够表明，传进来的闭包一定要返回一个布尔值，所以就可以把`return`忽略，而不引起歧义。


### 参数名称缩写

swift 自动为內联闭包提供参数名简称，可以通过`$0`,`$1`和`$2`去代表闭包的参数。
如果你用这些简写在你的闭包表达式中，就可以在闭包定义的时候忽略它的参数列表，`in`关键词可以就可以忽略了。

	reversed = names.sort({ $0 > $1 })

由于`String`本身就有自己的 > 方法，所以我们可以这样更简单的简化：

		reversed = names.sort( > )

### 尾随闭包

如果需要传递一个闭包表达式给一个函数，作为这个函数的最后参数，而这个闭包的表达式很长，就可以写成尾随闭包的形式。所谓尾随闭包就是一个闭包表达式写在函数调用的括号的最后面

不用尾随闭包的
	
	func someFunctionThatTakesAClosure(closure : () -> Void){
		//function body goes here
	}

	someFunctionThatTakesAClosure({
		//closure's body goes here
		})

用尾随闭包的

	someFunctionThatTakesAClosure(){
		//closure's body goes here
	}

	reversed = names.sort(){ $0 > $1 }

在调用的时候，也可以把后面的括号给忽略掉
	
	reversed = names.sort{ $0 > $1 }

### 捕获值(Capturing Values)

一个闭包可以捕获它定义时的上下文的一些变量和常量。在 swift 中最简单的捕获值形式就是在嵌套函数中能够获取它外部的参数和变量常量。

在捕获的时候，swift 会决定到底捕获的是引用还是值复制 ，简单说，如果在闭包中会对外部值改变就是一个引用的捕获，如果不会改变的话，就是一个值复制。而且 swift 会自动处理所有的内存管理。

	func makeIncrementer(forIncrement amout : Int) -> Void -> Int {
		var runningTotal = 0
		func incrementer() -> Int {
			runningTotal += amout
			return runningTotal
		}

		return incrementer
	}

这里的`runningTotal`就是引用捕获，`amout`就是值复制。

> Swift determines what should be captured by reference and what should be copied by value.You don't need to annotate `amount` or `runningTotal` to say that they can be used within the nested `incrementer` function.Swift also handles all memory mamagement involved in disposing of `runningTotal` when it is no longer needed by the `incrementer()` function.


### 闭包是引用类型（Closures Are Reference Types）

闭包的赋值都是一个引用。

	let incrementBySeven = makeIncrementer(forIncrement : 7)
	incrementBySeven()  

这里的`incrementBySeven`就是闭包的一个引用



### Reference

[The Swift Programming Language](https://itunes.apple.com/us/book/the-swift-programming-language/id881256329?mt=11)

[first class function](https://en.wikipedia.org/wiki/First-class_function)
