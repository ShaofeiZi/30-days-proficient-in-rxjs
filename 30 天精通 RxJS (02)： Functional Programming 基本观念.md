# 30 天精通 RxJS (02)： Functional Programming 基本观念

> 
> 
> Functional Programming 是 Rx 最重要的观念之一，基本上只要学会 FP 要上手 Rx 就不难了！Functional Programming 可以说是近年来的显学，各种新的函数编程语言推出之外，其他旧有的语言也都在新版中加强对 FP 的支援！
> 
> 

这是【30天精通 RxJS】的 02 篇，如果还没看过 01 篇可以往这边走：
[30 天精通 RxJS (01)： 认识 RxJS](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(01)%EF%BC%9A%E8%AE%A4%E8%AF%86%20RxJS.md)

## 什么是 Functional Programming ?

![functional programming icon](https://res.cloudinary.com/dohtkyi84/image/upload/v1481362001/cover/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7_2016-12-10_%E4%B8%8B%E5%8D%885.26.11_mgc7al.png)

Functional Programming 是一种编程范式(programming paradigm)，就像 Object-oriented Programming(OOP)一样，就是一种写程式的方法论，这些方法论告诉我们如何思考及解决问题。

简单说 Functional Programming 核心思想就是做运算处理，并用 function 来思考问题，例如像以下的算数运算式：

```
(5 + 6) - 1 * 3

```

我们可以写成

```
const add = (a, b) => a + b
const mul = (a, b) => a * b
const sub = (a, b) => a - b

sub(add(5, 6), mul(1, 3))

```

我们把每个运算包成一个个不同的 function，并用这些 function 组合出我们要的结果，这就是最简单的 Functional Programming。

## Functional Programming 基本要件

跟 OOP 一样不是所有的语言都支持 FP，要能够支持 FP 的语言至少需要符合**函数为一等公民**的特性。

### 函数为一等公民 (First Class)

一等公民就是指跟其他资料型别具有同等地位，也就是说函数能够被赋值给变数，函数也能够被当作参数传入另一个函数，也可当作一个函数的回传值

**函数能够被赋值给变数**

```
var hello = function() {}

```

**函数能被当作参数传入**

```
fetch('www.google.com')
.then(function(response) {}) // 匿名 function 被传入 then()

```

**函数能被当作回传值**

```
var a = function(a) {
	return function(b) {
	  return a + b;
	}; 
	// 可以回传一个 function
}

```

## Functional Programming 重要特性

### Expression, no Statement

Functional Programming 都是 表达式 (Expression) 不会是 陈述式(Statement)。
基本区分表达式与陈述式：

**表达式** 是一个运算过程，一定会有返回值，例如执行一个 function

```
add(1,2)

```

*   陈述式 则是表现某个行为，例如一个 赋值给一个变数

```
a = 1;

```

> 
> 
> 有时候表达式也可能同时是合法的陈述式，这里只讲基本的判断方法。如果想更深入了解其中的差异，可以看这篇文章 [Expressions versus statements in JavaScript](http://www.2ality.com/2012/09/expressions-vs-statements.html)
> 
> 

由于 Functional Programming 最早就是为了做运算处理不管 I/O，而 Statement 通常都属于对系统 I/O 的操作，所以 FP 很自然的不会是 Statement。

> 
> 
> 当然在实务中不可能完全没有 I/O 的操作，Functional Programming 只要求对 I/O 操作限制到最小，不要有不必要的 I/O 行为，尽量保持运算过程的单纯。
> 
> 

### Pure Function

**Pure function 是指 一个 function 给予相同的参数，永远会回传相同的返回值，并且没有任何显著的副作用(Side Effect)**

举个例子：

```
var arr = [1, 2, 3, 4, 5];

arr.slice(0, 3); // [1, 2, 3]

arr.slice(0, 3); // [1, 2, 3]

arr.slice(0, 3); // [1, 2, 3]

```

这里可以看到 slice 不管执行几次，返回值都是相同的，并且除了返回一个值(value)之外并没有做任何事，所以 `slice` 就是一个 pure function。

```
var arr = [1, 2, 3, 4, 5];

arr.splice(0, 3); // [1, 2, 3]

arr.splice(0, 3); // [4, 5]

arr.slice(0, 3); // []

```

这里我们换成用 `splice`，因为 `splice` 每执行一次就会影响 `arr` 的值，导致每次结果都不同，这就很明显不是一个 pure function。

**Side Effect**

Side Effect 是指一个 function 做了跟本身运算返回值没有关系的事，比如说修改某个全域变数，或是修改传入参数的值，甚至是执行 `console.log` 都算是 Side Effect。

Functional Programming 强调没有 Side Effect，也就是 function 要保持纯粹，只做运算并返回一个值，没有其他额外的行为。

这里列举几个前端常见的 Side Effect，但不是全部

*   发送 http request
*   在画面印出值或是 log
*   获得使用者 input
*   Query DOM 事件

**Referential transparency**

前面提到的 pure function 不管外部环境如何，只要参数相同，函数执行的返回结果必定相同。这种不依赖任何外部状态，只依赖于传入的参数的特性也称为 引用透明(Referential transparency)

### 利用参数保存状态

由于最近很红的 Redux 使我能很好的举例，让大家了解什么是用参数保存状态。了解 Redux 的开发者应该会知 Redux 的状态是由各个 reducer 所组成的，而每个 reducer 的状态就是保存在参数中！

```
function countReducer(state = 0, action) {
// ...
}

```

如果你跟 Redux 不熟可以看下面递回的例子

```
function findIndex(arr, predicate, start = 0) {
    if (0 <= start && start < arr.length) {
        if (predicate(arr[start])) {
            return start;
        }
        return findIndex(arr, predicate, start+1);
    }
}
findIndex(['a', 'b'], x => x === 'b'); // 找数组中 'b' 的 index

```

这里我们写了一个 findIndex 用来找数组中的元素位置，我们在 `findIndex` 中故意多塞了一个参数用来保存当前找到第几个 index 的**状态**，这就是利用参数保存状态！

> 
> 
> 这边用到了递回，递回会不断的呼叫自己，制造多层 stack frame，会导致运算速度较慢，而这通常需要靠编译器做优化！
> 
> 

> 
> 
> 那 JS 有没有做递回优化呢？ 恭喜大家，ES6 提供了 [尾呼优化(tail call optimization)](http://www.2ality.com/2015/06/tail-call-optimization.html)，让我们有一些手法可以让递回更有效率！
> 
> 

## Functional Programming 优势

### 可读性高

当我们透过一系列的函数封装资料的操作过程，代码能变得非常的简洁且可读性极高，例如下面的例子

```
[9, 4].concat([8, 7]) // 合并数组
      .sort()  // 排序
      .filter(x => x > 5) // 过滤出大于 5 的

```

### 可维护性高

因为 Pure function 等特性，执行结果不依赖外部状态，且不会对外部环境有任何操作，使 Functional Programming 能更好的除错及撰写单元测试。

### 易于并行/平行处理

Functional Programming 易于做并行/平行(Concurrency/Parallel)处理，因为我们基本上只做运算不碰 I/O，再加上没有 Side Effect 的特性，所以较不用担心 deadlock 等问题。

## 今日小结

今天讲了 Functional Programming 的基本特性，及其优势。现在愈来愈多的 Library 用到了 FP 的观念，JS 也越来越多 Functional 的函数库，例如：Lodash, Underscore, lazy, Ramda。了解 FP 的基本观念有助于我们在学习其他 Library 更容易上手，也能使我们撰写出更好的代码，希望各位读者有所收获，若有任何疑问欢迎在下方留言给我！