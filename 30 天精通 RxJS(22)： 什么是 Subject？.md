# 30 天精通 RxJS(22): Subject 基本观念


终于进到了 RxJS 的第二个重点 Subject，不知道读者们有没有发现？ 我们在这篇文章之前的范例，每个 observable 都只订阅了一次，而实际上 observable 是可以多次订阅的

```javascript
var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

source.subscribe(observerA);
source.subscribe(observerB);

// "A next: 0"
// "B next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "A complete!"
// "B next: 2"
// "B complete!"

```

[JSBin](https://jsbin.com/fopime/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/kkxp8551/1/)

上面这段代码，分别用 observerA 跟 observerB 订阅了 source，从 log 可以看出来 observerA 跟 observerB 都各自收到了元素，但请记得这两个 observer 其实是**分开执行**的也就是说他们是完全独立的，我们把 observerB 延迟订阅来证明看看

```javascript
var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

source.subscribe(observerA);
setTimeout(() => {
    source.subscribe(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 0"
// "A next: 2"
// "A complete!"
// "B next: 1"
// "B next: 2"
// "B complete!"

```

[JSBin](https://jsbin.com/fopime/5/edit?js,console) | [JSFiddle](https://jsfiddle.net/kkxp8551/2/)

这里我们延迟一秒再用 observerB 订阅，可以从 log 中看出 1 秒后 observerA 已经印到了 1，这时 observerB 开始印却是从 0 开始，而不是接着 observerA 的进度，代表这两次的订阅是完全分开来执行的，或者说是每次的订阅都建立了一个新的执行。

这样的行为在大部分的情境下适用，但有些案例下我们会希望第二次订阅 source 不会从头开始接收元素，而是从第一次订阅到当前处理的元素开始发送，我们把这种处理方式称为组播(multicast)，那我们要如何做到组播呢？

## 手动建立 subject

或许已经有读者想到解法了，其实我们可以建立一个中间人来订阅 source 再由中间人转送资料出去，就可以达到我们想要的效果

```javascript
var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subject = {
    observers: [],
    addObserver: function(observer) {
        this.observers.push(observer)
    },
    next: function(value) {
        this.observers.forEach(o => o.next(value))    
    },
    error: function(error){
        this.observers.forEach(o => o.error(error))
    },
    complete: function() {
        this.observers.forEach(o => o.complete())
    }
}

subject.addObserver(observerA)

source.subscribe(subject);

setTimeout(() => {
    subject.addObserver(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "B next: 2"
// "A complete!"
// "B complete!"

```

[JSBin](https://jsbin.com/fopime/6/edit?js,console) | [JSFiddle](https://jsfiddle.net/kkxp8551/3/)

从上面的代码可以看到，我们先建立了一个实例叫 subject，这个实例具备 observer 所有的方法(next, error, complete)，并且还能 addObserver 把 observer 加到内部的清单中，每当有值送出就会遍历清单中的所有 observer 并把值再次送出，这样一来不管多久之后加进来的 observer，都会是从当前处理到的元素接续往下走，就像范例中所示，我们用 subject 订阅 source 并把 observerA 加到 subject 中，一秒后再把 observerB 加到 subject，这时就可以看到 observerB 是直接收 1 开始，这就是组播(multicast)的行为。

让我们把 subject 的 `addObserver` 改名成 `subscribe` 如下

```javascript
var subject = {
    observers: [],
    subscribe: function(observer) {
        this.observers.push(observer)
    },
    next: function(value) {
        this.observers.forEach(o => o.next(value))    
    },
    error: function(error){
        this.observers.forEach(o => o.error(error))
    },
    complete: function() {
        this.observers.forEach(o => o.complete())
    }
}

```

> 
> 
> 应该有眼尖的读者已经发现，subject 其实就是用了 Observer Pattern。但这边为了不要混淆 Observer Pattern 跟 RxJS 的 observer 就不再提及。这也是为什么我们在一开始讲 Observer Pattern 希望大家亲自实现的原因。
> 
> 

> 
> 
> RxJS 中的 Subject 确实是类似这样运行的，可以在[原始码](https://github.com/ReactiveX/rxjs/blob/master/src/Subject.ts#L60)中看到
> 
> 

虽然上面是我们自己手写的 subject，但运行方式跟 RxJS 的 Subject 实例是几乎一样的，我们把前面的代码改成 RxJS 提供的 Subject 试试

```javascript
var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subject = new Rx.Subject()

subject.subscribe(observerA)

source.subscribe(subject);

setTimeout(() => {
    subject.subscribe(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "B next: 2"
// "A complete!"
// "B complete!"

```

[JSBin](https://jsbin.com/fopime/7/edit?js,console) | [JSFiddle](https://jsfiddle.net/kkxp8551/4/)

大家会发现使用方式跟前面是相同的，建立一个 subject 先拿去订阅 observable(source)，再把我们真正的 observer 加到 subject 中，这样一来就能完成订阅，而每个加到 subject 中的 observer 都能整组的接收到相同的元素。

## 什么是 Subject?

虽然前面我们已经示范直接手写一个简单的 subject，但到底 RxJS 中的 Subject 的概念到底是什么呢？

首先 Subject 可以拿去订阅 Observable(source) 代表他是一个 Observer，同时 Subject 又可以被 Observer(observerA, observerB) 订阅，代表他是一个 Observable。

总结成两句话

*   Subject 同时是 Observable 又是 Observer
*   Subject 会对内部的 observers 清单进行组播(multicast)

> 
> 
> 补充： 没事不要看！其实 Subject 就是 Observer Pattern 的实例并且继承自 Observable。
> 
> 

## 今日小结

今天介绍了 RxJS 中的第二个重点 Subject，重点放在 Subject 主要的运行方式，以及概念上的所代表的意思，如果今天还不太能够吸收的读者不用紧张，后面我们会讲到 subject 的一些应用，到时候就会有更深的体会。

不知道今天读者么有没有收获呢？ 如果有任何问题，欢迎在下方留言给我。