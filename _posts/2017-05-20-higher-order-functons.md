---
title: swift 常用高阶函数
tags: 码农世界
description: swift的高阶函数怎么使用？什么是高阶函数
---

<blockquote>map</blockquote>

``` swift
var arr = [1, 2, 3]
//map函数是有返回值的，想要arr里面的值map过去需要arr重新接收新值
arr.map { (a : Int) -> Int in
    return a * 2
}
//这种写法只是尾随闭包的简写。。
arr = arr.map {
    $0 * 2
}
```

<blockquote>flatMap</blockquote>

``` swift
//floatMap函数可以降维
var arr1 = [[1, 2], [4, 5], [6, 7]]
var aaa = arr1.flatMap {
    $0
}
//floatMap函数去空作用,还会解包数据
var arr2 = ["swift",  nil, "java"]

var bbb = arr2.flatMap {
    $0
}

```

<blockquote>filter</blockquote>

``` swift
//filter 函数用作对容器内元素进行筛选
var arr3 = ["swift", "java", "oc", "python"]
arr3 = arr3.filter {
    guard $0.characters.count > 2 else {
        return false
    }
    return true
}

```


<blockquote>reduce</blockquote>

``` swift
//reduce 函数用作对容器内元素进行操作，将元素字符串拼接
var arr4 = [2, 3, 4, 5]
//arr4.reduce("", {
//    return $0 == "" ? String($1) : $0 + String($1)
//})
//用reduce求和
arr4.reduce(0) {
    return $0 + $1
}
// 将数组中的每个字符串用‘,’拼接
let arr5 = ["Objective-C", "Swift", "HTML", "CSS", "JavaScript"]

// $0表示计算后的结果, $1表示数组中的每一个元素
arr5.reduce("", {
    return $0 == "" ? $1 : $0 + "," + $1
})

```



