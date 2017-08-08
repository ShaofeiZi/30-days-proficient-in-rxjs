# 30 天精通 RxJS(23): Subject, BehaviorSubject, ReplaySubject, AsyncSubject

昨天我们介绍了 Subject 是什么，今天要讲 Subject 一些应用方式，以及 Subject 的另外三种变形。

## Subject

昨天我们讲到了 Subject 实际上就是 Observer Pattern 的实例，他会在内部管理一份 observer 的清单，并在接收到值时遍历这份清单并送出值，所以我们可以这样用 Subject

```javascript
var subject = new Rx.Subject();

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

subject.subscribe(observerA);
subject.subscribe(observerB);

subject.next(1);
// "A next: 1"
// "B next: 1"
subject.next(2);
// "A next: 2"
// "B next: 2"

```

[JSBin](https://jsbin.com/nazekem/1/edit?js,console)

这里我们可以直接用 subject 的 next 方法传送值，所有订阅的 observer 就会接收到，又因为 Subject 本身是 Observable，所以这样的使用方式很适合用在某些无法直接使用 Observable 的前端框架中，例如在 React 想对 DOM 的事件做监听

```javascript
class MyButton extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
        this.subject = new Rx.Subject();

        this.subject
            .mapTo(1)
            .scan((origin, next) => origin + next)
            .subscribe(x => {
                this.setState({ count: x })
            })
    }
    render() {
        return <button onClick={event => this.subject.next(event)}>{this.state.count}</button>
    }
}

```

[JSBin](https://jsbin.com/nazekem/6/edit?js,output) | [JSFiddle](https://jsfiddle.net/jLh8ham3/3/)

从上面的代码可以看出来，因为 React 本身 API 的关系，如果我们想要用 React 自订的事件，我们没办法直接使用 Observable 的 creation operator 建立 observable，这时就可以靠 Subject 来做到这件事。

Subject 因为同时是 observer 和 observable，所以应用面很广除了前面所提的之外，还有上一篇文章讲的组播(multicase)特性也会在接下来的文章做更多应用的介绍，这里先让我们来看看 Subject 的三个变形。

## BehaviorSubject

很多时候我们会希望 Subject 能代表当下的状态，而不是单存的事件发送，也就是说如果今天有一个新的订阅，我们希望 Subject 能立即给出最新的值，而不是没有回应，例如下面这个例子

```javascript
var subject = new Rx.Subject();

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

subject.subscribe(observerA);

subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB); // 3 秒后才订阅，observerB 不会收到任何值。
},3000)

```

以上面这个例子来说，observerB 订阅的之后，是不会有任何元素送给 observerB 的，因为在这之后没有执行任何 `subject.next()`，但很多时候我们会希望 subject 能够表达当前的状态，在一订阅时就能收到最新的状态是什么，而不是订阅后要等到有变动才能接收到新的状态，以这个例子来说，我们希望 observerB 订阅时就能立即收到 `3`，希望做到这样的效果就可以用 BehaviorSubject。

BehaviorSubject 跟 Subject 最大的不同就是 BehaviorSubject 是用来呈现当前的值，而不是单纯的发送事件。BehaviorSubject 会记住最新一次发送的元素，并把该元素当作目前的值，在使用上 BehaviorSubject 建构式需要传入一个参数来代表起始的状态，范例如下

```javascript
var subject = new Rx.BehaviorSubject(0); // 0 为起始值
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

subject.subscribe(observerA);
// "A next: 0"
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB); 
    // "B next: 3"
},3000)

```

[JSBin](https://jsbin.com/nazekem/7/edit?js,console) | JSFiddle

从上面这个范例可以看得出来 BehaviorSubject 在建立时就需要给定一个状态，并在之后任何一次订阅，就会先送出最新的状态。其实这种行为就是一种状态的表达而非单存的事件，就像是年龄跟生日一样，年龄是一种状态而生日就是事件；所以当我们想要用一个 stream 来表达年龄时，就应该用 BehaviorSubject。

## ReplaySubject

在某些时候我们会希望 Subject 代表事件，但又能在新订阅时重新发送最后的几个元素，这时我们就可以用 ReplaySubject，范例如下

```javascript
var subject = new Rx.ReplaySubject(2); // 重复发送最后 2 个元素
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

subject.subscribe(observerA);
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB);
    // "B next: 2"
    // "B next: 3"
},3000)

```

[JSBin](https://jsbin.com/nazekem/10/edit?js,console) |

可能会有人以为 `ReplaySubject(1)` 是不是就等同于 BehaviorSubject，其实是不一样的，BehaviorSubject 在建立时就会有起始值，比如 `BehaviorSubject(0)` 起始值就是 `0`，BehaviorSubject 是代表着状态而 ReplaySubject 只是事件的重放而已。

## AsyncSubject

AsyncSubject 是最怪的一个变形，他有点像是 operator `last`，会在 subject 结束后送出最后一个值，范例如下

```javascript
var subject = new Rx.AsyncSubject();
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

subject.subscribe(observerA);
subject.next(1);
subject.next(2);
subject.next(3);
subject.complete();
// "A next: 3"
// "A complete!"

setTimeout(() => {
    subject.subscribe(observerB);
    // "B next: 3"
    // "B complete!"
},3000)

```

[JSBin](https://jsbin.com/nazekem/edit?js,console) |

从上面的代码可以看出来，AsyncSubject 会在 subject 结束后才送出最后一个值，其实这个行为跟 Promise 很像，都是等到事情结束后送出一个值，但实务上我们非常非常少用到 AsyncSubject，绝大部分的时候都是使用 BehaviorSubject 跟 ReplaySubject 或 Subject。

~~我们把 AsyncSubject 放在大脑的深处就好~~

## 今日小结

今天介绍了 Subject 的一些应用方式，以及 BehaviorSubject, ReplaySubject, AsyncSubject 三个变形各自的特性介绍，不知道读者么是否有收获呢？ 如果有任何问题，欢迎在下方留言给我！