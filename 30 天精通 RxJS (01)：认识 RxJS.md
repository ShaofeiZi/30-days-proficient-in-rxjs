# 30 天精通 RxJS (01)：认识 RxJS

> 
> 
> RxJS 是笔者认为未来几年内会非常红的 Library，RxJS 提供了一套完整的非同步解决方案，让我们在面对各种非同步行为，不管是 Event, AJAX, 还是 Animation 等，我们都可以使用相同的 API (Application Programming Interface) 做开发。
> 
> 

![RxJS Logo](https://raw.githubusercontent.com/Reactive-Extensions/RxJS/master/logos/logo.png)

这是【30天精通 RxJS】的 01 篇，如果还没看过 00 篇可以往这边走：
[30 天精通 RxJS (00)： 关于本系列文章](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%E5%A4%A9%E7%B2%BE%E9%80%9ARXJS%EF%BC%8801%EF%BC%89%EF%BC%9A%E5%85%B3%E4%BA%8E.md)

在网页的世界存取任何资源都是非同步(Async)的，比如说我们希望拿到一个档案，要先发送一个请求，然后必须等到档案回来，再执行对这个档案的操作。这就是一个非同步的行为，而随着网页需求的复杂化，我们所写的 JavaScript 就有各种针对非同步行为的写法，例如使用 callback 或是 Promise 对象甚至是新的语法糖 async/await —— 但随着应用需求愈来愈复杂，撰写非同步的程式码仍然非常困难。

## 非同步常见的问题

*   竞态条件 (Race Condition)
*   内存泄漏 (Memory Leak)
*   复杂的状态 (Complex State)
*   例外处理 (Exception Handling)

### [Race Condition](https://goo.gl/GlNLYl)

每当我们对同一个资源同时做多次的非同步存取时，就可能发生 Race Condition 的问题。比如说我们发了一个 Request 更新使用者资料，然后我们又立即发送另一个 Request 取得使用者资料，这时第一个 Request 和第二个 Request 先后顺序就会影响到最终接收到的结果不同，这就是 Race Condition。

### [Memory Leak](https://en.wikipedia.org/wiki/Memory_leak)

Memory Leak 是最常被大家忽略的一点。原因是在传统网站的行为，我们每次换页都是整页重刷，并重新执行 JavaScript，所以不太需要理会内存的问题！但是当我们希望将网站做得像应用程式时，这件事就变得很重要。例如做 [SPA](https://en.wikipedia.org/wiki/Single-page_application) (Single Page Application) 网站时，我们是透过 JavaScript 来达到切换页面的内容，这时如果有对 DOM 注册监听事件，而没有在适当的时机点把监听的事件移除，就有可能造成 Memory Leak。比如说在 A 页面监听 body 的 scroll 事件，但页面切换时，没有把 scroll 的监听事件移除。

### Complex State

当有非同步行为时，应用程式的状态就会变得非常复杂！比如说我们有一支付费用户才能播放的影片，首先可能要先抓取这部影片的资讯，接着我们要在播放时去验证使用者是否有权限播放，而使用者也有可能再按下播放后又立即按了取消，而这些都是非同步执行，这时就会各种复杂的状态需要处理。

### Exception Handling

JavaScript 的 try/catch 可以捕捉同步的例外，但非同步的程式就没这么容易，尤其当我们的非同步行为很复杂时，这个问题就愈加明显。

## 各种不同的 API

我们除了要面对非同步会遇到的各种问题外，还需要烦恼很多不同的 API

*   DOM Events
*   XMLHttpRequest
*   fetch
*   WebSockets
*   Server Send Events
*   Service Worker
*   Node Stream
*   Timer

上面列的 API 都是非同步的，但他们都有各自的 API 及写法！如果我们使用 RxJS，上面所有的 API 都可以透过 RxJS 来处理，就能用同样的 API 操作 (RxJS 的 API)。

这里我们举一个例子，假如我们想要监听点击事件(click event)，但点击一次之后不再监听。

**原生 JavaScript**

```
var handler = (e) => {
	console.log(e);
	document.body.removeEventListener('click', handler); // 结束监听
}

// 注册监听
document.body.addEventListener('click', handler);

```

**使用 Rx 大概的样子**

```
Rx.Observable
	.fromEvent(document.body, 'click') // 注册监听
	.take(1) // 只取一次
	.subscribe(console.log);

```

[JSbin](https://jsbin.com/vofaluv/4/edit?console,output) | [JSFiddle](https://jsfiddle.net/s6323859/d95a8peo/1/)
(点击画面后会在 console 显示，记得打开 console 来看)

大致上能看得出来我们在使用 RxJS 后，不管是针对 DOM Event 还是上面列的各种 API 我们都可以透过 RxJS 的 API 来做资料操作，像是范例中用 `take(n)` 来设定只取一次，之后就释放内存。

说了这么多，其实就是简单一句话

**在面对日益复杂的问题，我们需要一个更好的解决方法。**

## RxJS 基本介绍

RxJS 是一套借由 **Observable sequences** 来组合**非同步行为**和**事件基础**程序的 Library！

> 
> 
> 可以把 RxJS 想成处理 非同步行为 的 Lodash。
> 
> 

这也被称为 Functional Reactive Programming，更切确地说是指 Functional Programming 及 Reactive Programming 两个编程思想的结合。

> 
> 
> RxJS 确实是 Functional Programming 跟 Reactive Programming 的结合，但能不能称为 Functional Reactive Programming (FRP) 一直有争议。
> 
> 

> 
> 
> Rx 在[官网](http://reactivex.io/intro.html)上特别指出，有时这会被称为 FRP 但这其实是个“误称”。
> 
> 

> 
> 
> 简单说 FRP 是操作随着时间**连续性改变的数值** 而 Rx 则比较像是操作随着时间发出的**离散数值**，这个部份读者不用分得太细，因为 FRP 的定义及解释一直存在着歧异，也有众多大神为此争论，如下
> 
> 

> 
> 
> [André Staltz](https://medium.com/@andrestaltz/why-i-cannot-say-frp-but-i-just-did-d5ffaa23973b#.dhmsyic9w)：Rx 著名的推广者，也是 RxJS 5 主要贡献者之一，同时是 Cycle.js 的作者。Staltz 特别写了一篇[文章](https://medium.com/@andrestaltz/why-i-cannot-say-frp-but-i-just-did-d5ffaa23973b#.dhmsyic9w)解释为什么 Rx 不能说是 FRP 但他仍然称其为 FRP。
> 
> 

> 
> 
> [Juan Gomez](https://twitter.com/_juandg)：曾在 Netflix 工作，目前任职于 Fitbit，经常出现在国外演讨会，主要写 Android。Juan Gomez 在 [Droidcon NYC 2015 的演讲](https://realm.io/news/droidcon-gomez-functional-reactive-programming/)中特别提出他坚持称 Rx 为 FRP。
> 
> 

> 
> 
> [Evan Czaplicki](https://twitter.com/czaplic)：任职于 NoRedInk，Elm 的作者。Evan 在 [StrangeLoop 2014 的演讲](https://www.youtube.com/watch?v=Agu6jipKfYw)中，特别为现在各种 FRP 的不同解释做分类。
> 
> 

> 
> 
> 笔者自己的看法是比较偏向直接称 Rx 为 FRP，原因是这较为直觉(FP + RP = FRP)，也比较不会对新手造成困惑，另外就是其他各种编程范式(包含 OOP, FP)其实都是**想法的集合，而非严格的指南(Guideline)**，我们应该更宽松的看待 FRP 而不是给他一个严格的定义。
> 
> 

### 关于 Reactive Extension (Rx)

Rx 最早是由微软开发的 LinQ 扩展出来的开源专案，之后主要由社群的工程师贡献，有多种语言支援，也被许多科技公司所采用，如 Netflix, Trello, Github, Airbnb...等。

#### Rx 的相关资讯

*   开源专案 (Apache 2.0 License)
*   多种语言支持
    *   JavaScript
    *   Java
    *   C#
    *   Python
    *   Ruby
    *   ...(太多了列不完)
*   [官网](http://reactivex.io/)
*   ~~微软目前最成功的开源专案~~

> 
> 
> LinQ 唸做 Link，全名是 Language-Integrated Query，其功能很多元也非常强大；学 RxJS 可以不用会。
> 
> 

### Functional Reactive Programming

Functional Reactive Programming 是一种编程范式(programming paradigm)，白话就是一种**写程式的方法论**！举个例子，像 OOP 就是一种编程范式，OOP 告诉我们要使用对象的方式来思考问题，以及撰写程式。而 Functional Reactive Programming 其实涵盖了 Reactive Programming 及 Functional Programming 两种编程思想。

**Functional Programming**

Functional Programming 大部分的人应该多少都有接触过，这也是 Rx 学习过程中的重点之一，我们之后会花两天的篇幅来细讲 Functional Programming。
如果要用一句话来总结 Functional Programming，那就是 **用 function 来思考我们的问题，以及撰写程式**

> 
> 
> 在下一篇文章会更深入的讲解 Functional Programming
> 
> 

**Reactive Programming**

> 
> 
> 很多人一谈到 Reactive Programming 就会直接联想到是在讲 RxJS，但实际上 Reactive Programming 仍是一种编程范式，在不同的场景都有机会遇到，而非只存在于 RxJS，尤雨溪(Vue 的作者)就曾在 twitter 对此表达不满！
> 
> 

![Evan You 的推文](https://res.cloudinary.com/dohtkyi84/image/upload/v1480531809/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7_2016-11-30_%E4%B8%8A%E5%8D%886.23.49_zdniva.png)

Reactive Programming 简单来说就是 **当变数或资源发生变动时，由变数或资源自动告诉我发生变动了**

这句话看似简单，其实背后隐含两件事

*   当发生变动 => 非同步：不知道什么时候会发生变动，反正变动时要跟我说
*   由变数自动告知我 => 我不用写通知我的每一步程式码

由于最近很红的 Vue.js 底层就是用 Reactive Programming 的概念实作，让我能很好的举例，让大家理解什么是 Reactive Programming！

当我们在使用 vue 开发时，只要一有绑定的变数发生改变，相关的变数及画面也会跟着变动，而开发者不需要写这其中如何**通知**发生变化的每一步程式码，只需要**专注在发生变化时要做什么事**，这就是典型的 Reactive Programming (记得必须是由变数或资源主动告知！)

> 
> 
> Vue.js 在做 two-ways data binding 是透过 ES5 definedProperty 的 getter/setter。每当变数发生变动时，就会执行 getter/setter 从而收集有改动的变数，这也被称为**依赖收集**。
> 
> 

Rx 基本上就是上述的两个观念的结合，这个部份读者在看完之后的文章，会有更深的体悟。

## 今日小结

今天这篇文章主要是带大家了解为什么我们需要 RxJS，以及 RxJS 的基本介绍。若读者还不太能吸收本文的内容，可以过一段时间后再回来看这篇文章会有更深的体会，或是在下方留言给我！