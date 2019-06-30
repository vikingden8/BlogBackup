---
title: JavaScript中的var，let和const
date: 2019-6-30 22:15:08
tags:
categories: "JavaScript学习笔记"
---

这篇文章你将会学到在JavaScript (ES6)中创建变量的两种新方式，let和const。同时，我们会看下var，let以及const的差别。

ES2015 (or ES6)中介绍了两种创建变量的方式，let和const。在我们真正深入理解var，let和const的差别之前，有一些知识需要先了解。它们是变量声明和初始化，作用域（特别是函数作用域），以及变量提升。

### 变量声明 vs 变量初始化

```
var declaration
```

上面我们创建了一个新的声明。在JavaScript中，变量在创建的时候初始化的值为undefined。也就是说当我们尝试打印declaration的时候，输出会是undefined。

```
var declaration

console.log(declaration) // undefined
```

相对于变量声明，变量初始化就是在创建时指定了变量的值。

```
var declaration

console.log(declaration) // undefined

declaration = 'This is an initialization'
```

上面的例子，我们给变量declaration指定了一个字符值。下面我们看下作用域。

### 作用域

作用域定义了你程序内部的变量和函数的可访问性。在JavaScript中，有两种作用域：全局作用域以及函数作用域。

当你在函数中创建了一个以var的变量，那么它将只能在函数内部或者它的内部函数具有可访问性。

```
function getDate () {
  var date = new Date()

  return date
}

getDate()
console.log(date) // ❌ Reference Error
```

上面的例子中，当我们试图在函数外部访问函数中创建的变量时，会出现异常。这是因为date的作用域在getDate函数内部，只有函数本身以及内部函数才具有可访问性。

```
function getDate () {
  var date = new Date()

  function formatDate () {
    return date.toDateString().slice(4) // ✅
  }

  return formatDate()
}

getDate()
console.log(date) // ❌ Reference Error

```

我们再看一个高级的例子，假如有一价格集合 prices ，我们需要有这样的一个函数方法，其函数的参数就是价格集合以及折扣 discount，此函数返回一个打折后的价格集合。

```
discountPrices([100, 200, 300], .5) // [50, 100, 150]
```

实现可能是如下的样子：

```
function discountPrices (prices, discount) {
  var discounted = []

  for (var i = 0; i < prices.length; i++) {
    var discountedPrice = prices[i] * (1 - discount)
    var finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  return discounted
}
```

看起来很简单的例子，但是我们看一下for循环。在for循环中声明的变量是否在外部能被访问呢？结果是：可以的。

```
function discountPrices (prices, discount) {
  var discounted = []

  for (var i = 0; i < prices.length; i++) {
    var discountedPrice = prices[i] * (1 - discount)
    var finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

如果JavaScript是你唯一知道的语言，那么你可能对此没有任何的疑问。但是，如果你知晓其他的语言，如Java，那么你可能会有疑问，为什么会出现这样？毕竟，在for循环之后没有任何的原因需要再去访问变量i，discountedPrice以及finalPrice。那到底为啥会导致上述的问题呢？

### 变量提升

文章开头的时候说过：在JavaScript中，变量在创建的时候初始化的值为undefined。这就是变量提升。JavaScript解释器会在变量的创建阶段分配默认的undefined。

再看一下上一示例，变量提升是如何作用的。

```
function discountPrices (prices, discount) {
  var discounted = undefined
  var i = undefined
  var discountedPrice = undefined
  var finalPrice = undefined

  discounted = []
  for (var i = 0; i < prices.length; i++) {
    discountedPrice = prices[i] * (1 - discount)
    finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

需要注意的是，在函数方法的开头，所有的变量在声明时都被赋予了初始化值undefined。那也就是为什么尝试在真正声明变量之前去访问的时候，你会得到undefined的原因。

```
function discountPrices (prices, discount) {
  console.log(discounted) // undefined

  var discounted = []

  for (var i = 0; i < prices.length; i++) {
    var discountedPrice = prices[i] * (1 - discount)
    var finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

现在你知道如何使用var，最后我们来看下这篇文章的主旨，var，let以及const的区别到底在哪？

### var VS let VS const

#### var VS let

首先，我们先来看下var和let的区别。var和let的主要区别是：var是函数作用域级别，而let则是块作用域级别。也就是说，let关键字创建的变量只在块或块中内嵌的块具有可访问性。请注意，这里所说的块，就是用大花括号{}包起来的代码块，例如for循环，if语句等。

我们再来看下价格折扣的示例：

```
function discountPrices (prices, discount) {
  var discounted = []

  for (var i = 0; i < prices.length; i++) {
    var discountedPrice = prices[i] * (1 - discount)
    var finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

之前我们使用var关键字声明的i, discountedPrice, 以及finalPrice在for循环之后是可以打印出结果的。但是现在，我们把var关键字的变量全部改为let关键字，会出现什么样的结果呢？

```
function discountPrices (prices, discount) {
  let discounted = []

  for (let i = 0; i < prices.length; i++) {
    let discountedPrice = prices[i] * (1 - discount)
    let finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}

discountPrices([100, 200, 300], .5) // ❌ ReferenceError: i is not defined
```

从上可以看出，我们得到了一个错误：ReferenceError: i is not defined。这也就是说，用let关键字声明的变量是块作用域，而不是函数作用域。所以当我们尝试访问i (或 discountedPrice 或 finalPrice) 的时候，会得到引用错误的原因。

在之前的例子中，当我们在函数内部，使用var关键字声明的变量前面尝试打印变量时会得到undefined。

```
function discountPrices (prices, discount) {
  console.log(discounted) // undefined

  var discounted = []

  for (var i = 0; i < prices.length; i++) {
    var discountedPrice = prices[i] * (1 - discount)
    var finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

其实，在变量声明之前会访问变量，我是不能想到任何的这种使用场景。如何这样做的话，它应当抛出一个引用错误，而不是返回undefined。实际上，这也就是关键字let的作用。当你尝试在使用let关键字声明的变量之前去访问它时，你会得到一个引用错误ReferenceError。

```
function discountPrices (prices, discount) {
  console.log(discounted) // ❌ ReferenceError

  let discounted = []

  for (let i = 0; i < prices.length; i++) {
    let discountedPrice = prices[i] * (1 - discount)
    let finalPrice = Math.round(discountedPrice * 100) / 100
    discounted.push(finalPrice)
  }

  console.log(i) // 3
  console.log(discountedPrice) // 150
  console.log(finalPrice) // 150

  return discounted
}
```

#### let VS const

现在你理解了var和let的区别，那const呢？其实，const基本上类似于let。只是，唯一的差别就是一旦你给const关键字声明的变量赋予了值的话，则你不能再重新赋值：

```
let name = 'Tyler'
const handle = 'tylermcginnis'

name = 'Tyler McGinnis' // ✅
handle = '@tylermcginnis' // ❌ TypeError: Assignment to constant variable.
```

上面的例子用，用let关键字修饰的变量可以被重新赋值，但是用const关键字修饰的变量却不能。

现在我们最重要的问题还没有解答，那就是你应当使用 var，let还是const？最大众的回答就是，**你应当总是使用const，除非你知道变量需要被修改，则需要修改的变量应该使用let关键字。**
