# 30 天精通 RxJS(24): Observable operators - multicast, refCount, publish, share

昨天我们介绍完了各种 Subject，不晓得各位读者还记不记得在一开始讲到 Subject 时，是希望能够让 Observable 有新订阅时，可以共用前一个订阅的执行而不要从头开始，如下面的例子

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

上面这段代码我们用 subject 订阅了 source，再把 observerA 跟 observerB 一个个订阅到 subject，这样就可以让 observerA 跟 observerB 共用同一个执行。但这样的写法会让代码看起来太过复杂，我们可以用 Observable 的 multicast operator 来简化这段代码

## Operators

### multicast

multicast 可以用来挂载 subject 并回传一个可连结(connectable)的 observable，如下

```javascript
var source = Rx.Observable.interval(1000)
             .take(3)
             .multicast(new Rx.Subject());

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

source.subscribe(observerA); // subject.subscribe(observerA)

source.connect(); // source.subscribe(subject)

setTimeout(() => {
    source.subscribe(observerB); // subject.subscribe(observerB)
}, 1000);

```

[JSBin](https://jsbin.com/tehuhal/1/edit?js,console) | JSFiddle

上面这段代码我们透过 multicast 来挂载一个 subject 之后这个 observable(source) 的订阅其实都是订阅到 subject 上。

```javascript
source.subscribe(observerA); // subject.subscribe(observerA)

```

必须真的等到 执行 `connect()` 后才会真的用 subject 订阅 source，并开始送出元素，如果没有执行 `connect()` observable 是不会真正执行的。

```javascript
source.connect();

```

另外值得注意的是这里要退订的话，要把 `connect()` 回传的 subscription 退订才会真正停止 observable 的执行，如下

```javascript
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject()); // 无限的 observable 

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

var subscriptionA = source.subscribe(observerA);

var realSubscription = source.connect();

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe();
    subscriptionB.unsubscribe(); 
    // 这里虽然 A 跟 B 都退订了，但 source 还会继续送元素
}, 5000);

setTimeout(() => {
    realSubscription.unsubscribe();
    // 这里 source 才会真正停止送元素
}, 7000);

```

[JSBin](https://jsbin.com/tehuhal/2/edit?js,console) | JSFiddle

上面这段的代码，必须等到 `realSubscription.unsubscribe()` 执行完，source 才会真的结束。

虽然用了 multicast 感觉会让我们处理的对象少一点，但必须搭配 connect 一起使用还是让代码有点复杂，通常我们会希望有 observer 订阅时，就立即执行并发送元素，而不要再多执行一个方法(connect)，这时我们就可以用 refCount。

### refCount

refCount 必须搭配 multicast 一起使用，他可以建立一个只要有订阅就会自动 connect 的 observable，范例如下

```javascript
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject())
             .refCount();

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

var subscriptionA = source.subscribe(observerA);
// 订阅数 0 => 1

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
    // 订阅数 0 => 2
}, 1000);

```

[JSBin](https://jsbin.com/tehuhal/4/edit?js,console) | JSFiddle

上面这段代码，当 source 一被 observerA 订阅时(订阅数从 0 变成 1)，就会立即执行并发送元素，我们就不需要再额外执行 connect。

同样的在退订时只要订阅数变成 0 就会自动停止发送

```javascript
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject())
             .refCount();

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

var subscriptionA = source.subscribe(observerA);
// 订阅数 0 => 1

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
    // 订阅数 0 => 2
}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe(); // 订阅数 2 => 1
    subscriptionB.unsubscribe(); // 订阅数 1 => 0，source 停止发送元素
}, 5000);

```

[JSBin](https://jsbin.com/tehuhal/5/edit?js,console) | JSFiddle

### publish

其实 `multicast(new Rx.Subject())` 很常用到，我们有一个简化的写法那就是 publish，下面这两段代码是完全等价的

```javascript
var source = Rx.Observable.interval(1000)
             .publish() 
             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();

```

加上 Subject 的三种变形

```javascript
var source = Rx.Observable.interval(1000)
             .publishReplay(1) 
             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.ReplaySubject(1)) 
//             .refCount();

```

```javascript
var source = Rx.Observable.interval(1000)
             .publishBehavior(0) 
             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.BehaviorSubject(0)) 
//             .refCount();

```

```javascript
var source = Rx.Observable.interval(1000)
             .publishLast() 
             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.AsyncSubject(1)) 
//             .refCount();

```

### share

另外 publish + refCount 可以在简写成 share

```javascript
var source = Rx.Observable.interval(1000)
             .share();

// var source = Rx.Observable.interval(1000)
//             .publish() 
//             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();

```

## 今日小结

今天主要讲解了 multicast 和 refCount 两个 operators 可以帮助我们既可能的简化代码，并同时达到组播的效果。最后介绍 publish 跟 share 几个简化写法，这几个简化的写法是比较常见的，在理解 multicast 跟 refCount 运作方式后就能直接套用到 publish 跟 share 上。

不知道今天读者们有没有收获呢？ 如果有任何问题欢迎在下方留言给我，谢谢＾＾