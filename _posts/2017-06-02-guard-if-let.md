---
title: 可选绑定、guard else weak unowned
tags: 码农世界
description: swift的可选绑定和guard的用法
---

这几个关键字放一起也是醉了，本来想放到上一篇一起，想了想放哪里好像都一样。主要是去了解一下这些语句（关键字）的作用以及使用场景及优势。


```swift
func sayHello(name:String?) {
    if name == nil {
        print("姓名是空的")
    } else {
        print("hello world" + " " + name!)
    }
}
sayHello(name: nil)

func sayHello1(name:String?) {
    if let tmpName = name {
        print("hello world" + " " + tmpName)
    }
    print("姓名是空的")
}
sayHello1(name: "zhangsan")

func sayHello2(name:String?) {
    guard let tmpName = name  else {
        print("姓名是空的 from guard")
        return
    }
    print("hello world" + " " + tmpName)
}
sayHello2(name: "zhangsan")


//defer 推迟到最后执行
func sayHello3() {
    print("first")
    defer {
        print("defer statement")
    }
    print("last")
}
sayHello3()

```
>结论

可选绑定本质和if else没区别，好处在于会解包数据 ，在之后使用不必要再去解包，guard else 和可选绑定也非常相似，可以在else里面抛异常或者return。 defer推迟到最后执行，这个得到具体场景具体使用。


>weak

```swift
class Person {
    var name : String?
    var book : Book?
    deinit {
        print("person release")
    }
}

class Book {
    var name : String?
    weak var person : Person?
    deinit {
        print("book release")
    }
}

var person : Person? = Person()
person?.book = Book()
person = nil
```
person 强引用book，而book弱引用person，所以在person被释放时，book也没有任何强引用而释放，达到解除循环引用的问题，同时weak标记的对象，在释放之后会自动置为nil，避免了野指针问题。

>unowned

```swift
class Person {
    var name : String?
    var book : Book?
    
    init(name:String?) {
        self.name = name
        self.book = Book(name: "", person: self)
    }
    
    deinit {
        print("person release")
    }
}

class Book {
    var name : String?
    unowned let person : Person
    
    init(name:String?, person:Person) {
        self.name = name
        self.person = person
    }
    
    deinit {
        print("book release")
    }
}

var person : Person? = Person(name: "zhangsan")
person = nil
```

无主引用和weak一样可以打破循环引用，区别在于无主引用通常都被期望拥有值， 无法在实例被销毁后将无主引用设为nil，因为非可选类型的变量不允许被赋值为nil。具体引用计数请前往[这里](http://wiki.jikexueyuan.com/project/swift/chapter2/16_Automatic_Reference_Counting.html)
