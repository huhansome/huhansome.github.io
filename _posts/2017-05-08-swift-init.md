---
title: swift构造器，swift类的初始化方法
tags: 码农世界
description: swift类的初始化，swift init方法最全面的介绍
---

swift里面会经常看到申明属性申明成可选类型，其实这和构造器也是有关系的。(申明变量时，用？申明可选值类型的话，在之后使用时都需要显式的写上解包操作，而用！申明时，之后使用会默认隐式的加上！解包，两者申明之后不初始化都会默认一个nil值)

>构造器

```swift
class Student:NSObject {
    var name:String?
    var age:Int?
    var height:Double!//申明成可选值类型，不初始化会默认nil值，用！在以后使用不需要解包操作（默认隐式加上了）
    var weight:Double?
    
    init(name:String?) {  //指定构造器
        super.init()
        self.name = name
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
    
    init(name:String?, height:Double?) {
        super.init()
        self.name = name
        self.height = height
    }
    
    convenience init(name:String?, age:Int?, weight:Double?, height:Double?) { //便利构造器里面不能调用super,便利构造器必须调用指定构造器去初始化
        self.init(name: name, height: height)
        self.age = age
        self.weight = weight
    }
}
let student = Student() //由于有指定构造器，系统默认的默认构造器没了，以至于报错
```
>总结
>当没有自定义构造器且属性都有默认值的时候，会有默认构造器。当有指定构造器时，默认构造器会不能用。便利构造器必须要调用指定构造器来完成初始化，便利构造器不能调用super。关于构造器更多信息请前往[这里](http://wiki.jikexueyuan.com/project/swift/chapter2/14_Initialization.html#default_initializers)
