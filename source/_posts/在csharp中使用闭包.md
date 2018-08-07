---
title: 在C#中使用闭包
date: 2018-08-01 11:50:08
tags: 
- csharp
- base 
---
> 本文不可避免地会出现以下概念，希望在阅读前能理解：
- 闭包(Closure)
- C# 中的委托，lambda表达式 ，匿名方法

# 前言
## javascript中的闭包
这个概念经常在javascript中见到，因为闭包几乎是支撑起javascript生态不可或缺的重要特性！在你熟悉的任何库或框架中（如jquery，vue等）都使用了闭包。

在其他强类型的语言如java, C#，私有变量可以通过`private`修饰一个成员变量声明。   
但是在js中没有这种机制，但是同样可通过闭包封装私有变量：利用了闭包保存了内部函数变量的引用，而外部函数无法直接获取变量的引用。

比如，在浏览器中原生只有 "click" 事件可供使用，并不提供 "双击事件"，利用闭包，我们可以轻松实现双击：
```js
// 双击包装函数
    // 闭包保存了如下变量的引用，形成"私有变量"
function dblclick(handler){
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


# C#中的闭包
C#中的闭包也是同样的表现：通过闭包保存了内部函数对变量的引用。

## 一个经典的案例
声明一个集合，保存我们循环5次期间所创建的函数，每个函数打印出当前循环的索引值。
```csharp
List<Action> actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(
        () => Console.WriteLine(i)
    );
}

actions.ForEach(func => func()); 
// 输出：
// 5
// 5
// 5
// 5
// 5
```
所有的变量都打印为最终的索引，因为打印的变量 i 的值指向了最终的索引值。

很显然这不是我们想要的结果，所以在每次循环的时候，我们需要保存 i 的值，这个时候就轮到 "闭包" 发挥作用了。

```csharp
// 创建一个函数，接收一个值
// 传入的索引值和返回函数就形成了闭包
Action ActionFactory(int index)
{
    return () => Console.WriteLine(index);
}


List<Action> actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(
        ActionFactory(i)
    );

}
actions.ForEach(func => func());
// 输出：
// 0
// 1
// 2
// 3
// 4
```
传入`ActionFactory`的`index`参数被内部函数所引用，形成闭包，结果就是我们想要的。

## 创建闭包
以上演示可以看出，闭包实质上就是保存一个变量在内部函数的引用，而外部函数无法访问。  
所以在C#中可创建函数形成闭包的方式有：
- 委托
- 表达式树

举一个简单的例子：  
创建一个函数，根据传入参数值，返回一个附加该值的函数
### 使用 委托 创建
```csharp
// 通过Lambda
Func<int,int> SetAdditional(int additional)
{
    return v => (additional + v);
}

// 通过匿名方法
Func<int,int> SetAdditional(int additional)
{
    return delegate (int v)
    {
        return additional + v;
    };
}


// 获取闭包后的函数
var addition_10 = SetAdditional(10);
var addition_1 = SetAdditional(1);

var num =  1;
addition_1(num);  // 2
addition_10(num); // 11
```
### 使用 表达式树 创建
 表达式树也是通过创建委托的方式形成闭包。
```csharp
Func<int, int> SetAdditional(int additional)
{
    var parameter = Expression.Parameter(typeof(int), "v");
    var addition = Expression.Constant(additional,typeof(int));

    var lambda = Expression.Lambda(
        Expression.Add(
            addition,
            parameter),
        parameter);

    return lambda.Compile() as Func<int,int>;
}

// 获取闭包后的函数
var addition_10 = SetAdditional(10);
var addition_1 = SetAdditional(1);

var num =  1;
addition_1(num);  // 2
addition_10(num); // 11
```

>⚠️ 一般函数执行完毕后，内部的变量会被回收。但是由于闭包保存了变量的引用，导致该变量无法被释放，始终占用物理内存。  
当闭包的函数越来越多的时候，就造成了程序的性能下降

----
闭包是委托的一种使用技巧，而委托能使得代码更具抽象能力。正确的理解闭包无疑可以更好的使用委托，使代码更"优雅"。