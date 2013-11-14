---
layout: post
title: "Unit Test Result Macro Reference"
description: "some macro for ios unit- test"
category: "IOS unit-test"
tags: [ios]
---

So far,I have never do Unit-Test .This is the beginning for to learn about Unit-Test on ios develope .If you find something wrong ,pls corret it ,I will appreciate it.

As the title,this artitle is about Unit-Test Result Macro ,which we use on unit test.For more details,pls go to apple's [document](https://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/UnitTesting/00-About_Unit_Testing/about.html)

Unconditional Failure
------------------------
### STFail
Description of the failure.

`STFail(failure_description, ...)`

###### Parameters 

*failure_description*:specify the error message.

*...*  :(Optional) A comma-separated list of arguments to substitute into *failure_description*

Equality Tests
-----------------------
### Equality Tests

`STAssertEqualObjects(object_1, object_2, failure_description, ...)`

Fail when `[object1 isEquelTo:object2]`

### STAssertEquals
`STAssertEquals(value_1, value_2, failure_description, ...)`

Fail when `value1` is not equal to `value2`

### STAssertEqualsWithAccuracy
`STAssertEqualsWithAccurary(value1,value2,accuracy,failure_description,...)`

Fail when the differenc between `value1` and `value2` is greater than `accuracy`,`value1`,`value2` and `accuracy` are integer or floating-point value


Nil Tests
---------------------------
### STAssertNil
`STAssertNil(expression, failure_description, ...)`

Fail when a given expression is not nil

### STAssertNotNil
`STAssertNotNil(expression, failure_description, ...)`

Fail when a given expression is nil.

Boolean Tests
-----------------------
### STAssertTrue
`STAssertTrue(expression, failure_description, ...)`

Fail when a given expression is false

### STAssertFalse
`STAssertFalse(expression, failure_description, ...)`

Fail when a given expression is true

Exception Tests 
--------------------

### STAssertThrows
`STAssertThrows(expression, failure_description, ...)`

Fail when a given expression does't raise an exception

### STAssertThrowSpecific 
`STAssertThrowsSpecific(expression, exception_class, failure_description, ...)`

Fail when a given expression does't raise an exception of a particular class

### STAssertThrowSpecificNamed
`STAssertThrowsSpecificNamed(expression, exception_class, exception_name, failure_description, ...)`

Fail when a given expression does't raise an exception of a particular class with a given name.

### STAssertNoThrow
`STAssertNoThrow(expression, failure_description, ...)`

Fail when a given expression raise an exception

### STAssertNoThrowSpecific
`STAssertNoThrowSpecific(expression, exception_class, failure_description, ...)`

Fail when a given expression raises an exception of the exception_class

### STAssertNoThrowSpecificNamed
`STAssertNoThrowSpecificNamed(expression, exception_class, exception_name, failure_description, ...)`

Fail when a given expression raise an exception of the class exception_class with the name exception_name

### STAssertTrueNoThrow
`STAssertTrueNoThrow(expression, failure_description, ...)`

Fail when a given expression is false or raises an exception

### STAssertFalseNoThrow 
`STAssertFalseNoThrow(expression, failure_description, ...)`

Fail when a given expression is true or raise an exception.


