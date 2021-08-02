---
title: textView和textField输入控制
tags: 码农世界
description: swift协议的使用，UITextView和UITextField输入控制
---

#### InputInspectable

对于textView和textField输入常常会遇到很多的要求，现在的三方键盘越来越多，输入的乱七八糟的也多，对于检测是个挑战，而且很多时候因为输入了这些火星文导致程序异常，InputInspectable就是为了解决这类输入的问题，方便好用，不需要做修改，也不需要写大量代码，甚至订阅通知、设置代理等，只需要设置一个正则表达式即可达到目的，对于正则，代码中附带了多个正则，对于正则字符串的管理借鉴了NSNotification.Name的方式，方便扩展和使用。有了这样的想法之后就开始着手实现，有时候好像写个基类，让使用者继承会好写一点，但是感觉那样代码具有太强侵入性，对于人家使用也相对复杂，那么到底能不能面向切面，直接实现了呢？

swift协议非常强大，基于协议编程，很容易实现一些切面无侵入的代码，于是很快就写出了下面的代码

```swift
public protocol InputInspectable {
    var maxInput: Int = 0
    var inputRegExs = [InputRegEx.Name]()
    var historyText = ""
}
extension UITextField : InputInspectable {...}
extension UITextView : InputInspectable {...}
```

但是很快就发现了，协议里面的属性并不是存储属性，会导致很多问题，根本就达不到自己想要的效果，于是经过几番折腾之后搞出一个中间件来实现这些属性的存储，以及输入监听，让textView和textField遵守InputInspectable协议来实现扩展，于是出现下面的代码

```swift
public protocol InputInspectable {
    var input_compont: InputInspectableComponent{ get }
}
extension UITextField : InputInspectable {...}
extension UITextView : InputInspectable {...}
public class InputInspectableComponent {
    var maxInput: Int = 0
    var inputRegExs = [InputRegEx.Name]()
    var historyText = ""
    public init() {
        //...
    }
}
///不是存储属性
extension InputInspectable {
    public var compont: InputInspectableComponent {
        get {
            return InputInspectableComponent()
        }
        set {}
    }
}
```

但是这样会导致其他问题，每个textView、textField输入都会受到影响，这当然不是想要的，那么到底怎么实现中间件只管理我们想要管理的输入控件呢，于是要想办法把输入控件持有才行，但是一旦持有对于内存管理又会有问题，必须要解决内存问题，由于协议里面的属性不是存储属性，对于输入控件来说中间件也不是存储属性，于是利用runtime做一个模拟懒加载的操作，在中间件中用weak方式持有输入控件，达到目的

```swift
public class InputInspectableComponent<Base: AnyObject> {
    var maxInput: Int = 0
    var inputRegExs = [InputRegEx.Name]()
    var historyText = ""
    public unowned var base: Base
    public init(_ base: Base) {
        self.base = base
    }
}	

public protocol InputInspectable {
    associatedtype Base: AnyObject
    var input_compont: InputInspectableComponent<Base>? { get }
}

extension UITextField : InputInspectable {
    private struct AssociatedKeys {
        static var componetKey: String = "TextFiled_componetKey"
    }
    
    public var input_compont: InputInspectableComponent<TextField>? {
        get {
            guard let c = objc_getAssociatedObject(self, &AssociatedKeys.componetKey) as? InputInspectableComponent<TextField> else {
                let cc = InputInspectableComponent<TextField>(self)
                objc_setAssociatedObject(self, &AssociatedKeys.componetKey, cc, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
                return cc
            }
            return c
        }
        set {
            objc_setAssociatedObject(self, &AssociatedKeys.componetKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}

extension UITextView : InputInspectable {
    private struct AssociatedKeys {
        static var componetKey: String = "TextView_componetKey"
    }
    public var input_compont: InputInspectableComponent<TextView>? {
        get {
            guard let c = objc_getAssociatedObject(self,  &AssociatedKeys.componetKey) as? InputInspectableComponent<TextView> else {
                let cc = InputInspectableComponent<TextView>(self)
                objc_setAssociatedObject(self, &AssociatedKeys.componetKey, cc, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
                return cc
            }
            return c
        }
        set {
            objc_setAssociatedObject(self, &AssociatedKeys.componetKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}
```

这样只会就能解决内存问题，也能达到管理我们指定的输入控件，而且可以说无侵入，使用的时候只要调用属性使用，如果不需要控制就不调用属性即可：`textView.input_compont?.maxInput = maxInput;`
`t.input_compont?.inputRegExs = [InputRegEx.Name.chineseOnly];` 另外对于正则管理也采用了`NSNotification.Name`类似的方式管理，而不是满篇的字符串横飞。

[项目源码](https://github.com/CheaterHu/InputInspectable)