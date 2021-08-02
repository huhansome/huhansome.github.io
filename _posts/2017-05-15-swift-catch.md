---
title: swift异常处理，swift的try catch的正确使用姿势
tags: 码农世界
description: 看完这篇精通swift的try catch
---


程序健壮性很重要，要是一直在崩溃，给用户的印象真的是致命的。好在有异常捕获，有一些救治措施。

看代码，学习swift的异常捕获


```swift
enum MyError : Error {
    case empty(String)
    case tooBig(String)
}

class Student:NSObject {
    var name:String?
    var age:Int?
    var height:Double!//申明成可选值类型，不初始化会默认nil值，用！在以后使用不需要解包操作（默认隐式加上了）
    var weight:Double?
    
    init(name:String?) {  //指定构造器
        super.init()
        self.name = name
    }

    init(name:String?, height:Double?) {
        super.init()
        self.name = name
        self.height = height
    }
    
    init(name:String?, age:Int?, weight:Double?) throws { //指定构造器
        super.init()
        
        guard name != nil else {
            throw MyError.empty("姓名不能为空")
        }
        self.name = name
        self.age = age
        self.weight = weight
    }
    
    convenience init(name:String?, age:Int?, weight:Double?, height:Double?) throws { //便利构造器里面不能调用super,便利构造器必须调用指定构造器去初始化
        do {
            try self.init(name: name, age:age, weight:weight)
        } catch let error as MyError {
            throw error
        }
//        //直接try!会在错误时崩溃，打破错误传播链
//        try! self.init(name: name, age:age, weight:weight)
        self.height = height
    }
}

// 调用便利构造器，让程序异常,捕获异常
do {
    let student1 = try Student(name: nil, age: 18, weight: 38, height: 60)
} catch MyError.empty(let errorString) {
    print(errorString)
}


//try? 会在出错时，返回nil，不会导致程序崩溃，如果没有错误，会返回一个可选值
//let s = try? Student(name: nil, age: 13, weight: 34)


//try! 会在出错时崩溃
//let s = try! Student(name: nil, age: 13, weight: 34)
```
>结论

try? 会在出错时，返回nil，不会导致程序崩溃，如果没有错误，会返回一个可选值<br>
try! 会在出错时崩溃，打破错误传播链<br>
为了程序健壮性，建议使用docatch
