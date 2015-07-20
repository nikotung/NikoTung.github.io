---
layout: post
title: "swift学习笔记 -- optional value ? and !"
description: "swift学习笔记"
category: "2015-07"
tags: [optional value,swift]
---

在swift中，你可以通过`var`关键词去声明一个变量，通过`let`去声明一个常量。与OC一个不一样的地方是，在OC中声明一个变量的时候，如果没有赋默认值则就是一个默认的`nil`，直接去使用也不会报错。而在swift中你没有初始化一个变量，而去直接使用是会报错的，而且还是在编译期就报错了，因为swift中不会在声明一个对象的时候有一个`nil`的默认值。

在swift中还引入了一个叫**optionals**的这样的变量。你可以这么理解：
	
* 它有一个值，并且等于x
* 或者它根本就没有一个值

表明它可能是一个缺省值。这种概念在OC中不存在的，而且也跟OC中跟它比较接近的nil也不一样。在OC中nil是一个指向空对象的指针,所以只能用于对象，而在swift中应该是一个语意上的概念，能用于任何对象。

可以通过下面的方法，在变量的类型后面添加一个？去标明改变量是一个`optional`的值(变量与?间不能有空格)。
	
	var value : String? //nil
	value?.hashValue // nil
	var optional : String? = "hash"  //hash
	optional?.hashValue    //4799450059810186505

如果你在声明的时候没有给`optional`赋初值，那么它就默认为nil(跟变量不一样的)

有一个地方需要注意的是?其实就是一个语法糖。`optional`其实是一个`enum`。

	
	enum Optional<T> : _Reflectable, NilLiteralConvertible {
    	case None
    	case Some(T)

	    /// Construct a `nil` instance.
	    init()

	    /// Construct a non-`nil` instance that stores `some`.
	    init(_ some: T)

	    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
	    func map<U>(@noescape f: (T) -> U) -> U?

	    /// Returns `nil` if `self` is nil, `f(self!)` otherwise.
	    func flatMap<U>(@noescape f: (T) -> U?) -> U?

	    /// Returns a mirror that reflects `self`.
	    func getMirror() -> MirrorType

	    /// Create an instance initialized with `nil`.
	    init(nilLiteral: ())
	}


它有两个类型，一个是`None`一个是`Some(T)`,这样应该就比较好理解`optional`的含义了。它有两个构造函数`init()`和`init(_ some : T)`。?的语法糖的作用就是这样

* 如果你在声明一个`optional`时，没有赋初值，那么它就调用了一个`init()`构造函数返回一个`nil`
* 如果你在声明一个`optional`时，赋了初值，那么它就调用了`init(_ some : T)`返回一个`T`类型的的实例

在访问`optional`变量时，可以通过if 语句去判断它是不是nil，(== nil 或者 != nil)
	
	if value != nil {
		print("the value variable contain some value")
	}

当你百分百确认`optinal`变量是有值的时候，你可以通过在变量后面添加感叹号!去访问 

	if optional != nil {
	    print("value of optional \(optional!)")
	}

这种就成为强制拆包，强行返回`optional`声明时的`T`类型数据。如果你用强制拆包!时，`optional`却还没有值，那么app就会crash了，触发了一个运行时的错误。

Apple建议使用一种叫`optinal binding`的方法去访问`optional`变量。主要就是通过if/while语句，去检查`optinal`并它取出来把它放到一个变量或者常量中，如果转换成功就会执行一下步，而且在这里面就不再需要用!去访问变量了。

	if let actualValue = value {
		print("the actual value is \(actualValue)")
	}

隐式拆包(implicitly unwrapped optional)。有时候，一个`optional`的值一旦被设好了，就会一直存在，这时候可以在声明的时候用!而不是？去修饰，在后续的使用过程中就可以直接访问了。这种比较常见的是用于一个类的属性中。

	let assumedString : String! = "an implicitly unwrapped optional" 
	print("the assumed string is \(assumedString)")


在函数/方法的调用中，一个返回optional的方法再去调用别的方法时，一旦返回optional的方法返回了nil，后面的调用都会返回nil。












