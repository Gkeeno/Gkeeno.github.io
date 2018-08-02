---
title: 在C#中使用闭包
date: 2018-08-01 11:50:08
tags: 
- C#
- base
---
> 本文不可避免地会出现以下概念，希望在阅读前能理解：
- javascript中的闭包
- C# 中的委托，lambda表达式 ，匿名方法

# javascript中的闭包(Closure)
这个概念经常在javascript中见到，因为闭包几乎是支撑起javascript生态不可或缺的重要特性！在你熟悉的任何库或框架中都或多或少使用了闭包，比如jquery，vue等。

在其他强类型的语言如java, C#，私有变量可以通过`private`修饰一个成员变量声明。   
但是在js中没有这种机制，但是同样可通过闭包封装私有变量：利用了闭包保存了内部函数变量的引用，而外部函数无法获取改变量的引用。

众所周知，在浏览器中原生只有"click"事件可供使用，并不提供双击，利用闭包，我们可以轻松实现双击：
```js
// 封装双击事件
function dblclick(handler){
    // 闭包保存了如下变量的引用，形成"私有变量"
    var delay = 300; //双击间隔最大300ms
    var first = Date.now();

    return function(){
        var second = Date.now();
        var isDblclick = second - first <= delay;
        first = second;
        if(isDblclick) handler();
    }
}

// 绑定点击事件
document
.querySelector("body")
.addEventListener("click",dblclick(function(){
    alert("dblclick!");
}))
```

## 一个经典的案例
```

```

# C#中的闭包



