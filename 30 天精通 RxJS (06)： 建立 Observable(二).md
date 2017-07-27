# 30 天精通 RxJS (06)： 建立 Observable(二)

> 
> 
> 通常我们会透过 creation operator 来建立 Observable 实例，这篇文章会讲解几个较为常用的 operator！
> 
> 

这是【30天精通 RxJS】的 06 篇，如果还没看过 05 篇可以往这边走：
[30 天精通 RxJS (05)： 建立 Observable(一)](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(05)%EF%BC%9A%20%E5%BB%BA%E7%AB%8B%20Observable(%E4%B8%80).md)

## Creation Operator (创建 运算符)

Observable 有许多创建实例的方法，称为 creation operator。下面我们列出 RxJS 常用的 creation operator

*   create
*   of
*   from
*   fromEvent
*   fromPromise
*   never
*   empty
*   throw
*   interval
*   timer

### of

还记得我们昨天用 `create` 来建立一个同步处理的 observable 吗？

```
var source = Rx.Observable
    .create(function(observer) {
        observer.next('Jerry');
        observer.next('Anna');
        observer.complete();
    });

source.subscribe({
    next: function(value) {
    	console.log(value)
    },
    complete: function() {
    	console.log('complete!');
    },
    error: function(error) {
    console.log(error)
    }
});

// Jerry
// Anna
// complete!

```

[JSBin](https://jsbin.com/xerori/2/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/)

他先后传递了 `'Jerry'`, `'Anna'` 然后结束(complete)，这是一个十分常见模式。当我们想要**同步的**传递几个值时，就可以用 `of` 这个 operator 来简洁的表达!

下面的代码行为同上

```
var source = Rx.Observable.of('Jerry', 'Anna');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

// Jerry
// Anna
// complete!

```

[JSBin](https://jsbin.com/xerori/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/1/)

是不是相较于原本的代码简洁许多呢？

### from

可能已经有人发现其实 `of` operator 的一个一个参数其实就是一个 list，而 list 在 JavaScript 中最常见的形式是阵列(array)，那我们有没有办法把一个已存在的阵列当作参数呢？

有的，我们可以用 `from` 来接收任何可列举的参数！

```
var arr = ['Jerry', 'Anna', 2016, 2017, '30 days'] 
var source = Rx.Observable.from(arr);

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

// Jerry
// Anna
// 2016
// 2017
// 30 days
// complete!

```

[JSBin](https://jsbin.com/xerori/4/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/2/)

记得任何可列举的参数都可以用喔，也就是说像 Set, WeakSet, Iterator 等都可以当作参数！

> 
> 
> 因为 ES6 出现后可列举(iterable)的型别变多了，所以 `fromArray` 就被移除囉。
> 
> 

另外 from 还能接收字串(string)，如下

```
var source = Rx.Observable.from('铁人赛');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});
// 铁
// 人
// 赛
// complete!

```

[JSBin](https://jsbin.com/xerori/5/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/3/)

上面的代码会把字串里的每个字一一印出来。

我们也可以传入 Promise 物件，如下

```
var source = Rx.Observable
  .from(new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('Hello RxJS!');
    },3000)
  }))

source.subscribe({
    next: function(value) {
    	console.log(value)
    },
    complete: function() {
    	console.log('complete!');
    },
    error: function(error) {
    console.log(error)
    }
});

// Hello RxJS!
// complete!

```

[JSBin](https://jsbin.com/gefisiy/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/d95a8peo/5/)

如果我们传入 Promise 物件实例，当正常回传时，就会被送到 next，并立即送出完成通知，如果有错误则会送到 error。

> 
> 
> 这里也可以用 `fromPromise` ，会有相同的结果。
> 
> 

### fromEvent

我们也可以用 Event 建立 Observable，透过 `fromEvent` 的方法，如下

```
var source = Rx.Observable.fromEvent(document.body, 'click');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

// MouseEvent {...}

```

[JSBin](https://jsbin.com/xerori/6/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/4/)

`fromEvent` 的第一个参数要传入 DOM 物件，第二个参数传入要监听的事件名称。上面的程式会针对 body 的 click 事件做监听，每当点击 body 就会印出 event。

> 
> 
> 取得 DOM 物件的常用方法：
> `document.getElementById()`
> `document.querySelector()`
> `document.getElementsByTagName()`
> `document.getElementsByClassName()`
> 
> 

**补充：fromEventPattern**

要用 Event 来建立 Observable 实例还有另一个方法 `fromEventPattern`，这个方法是给类事件使用。所谓的类事件就是指其行为跟事件相像，同时具有注册监听及移除监听两种行为，就像 DOM Event 有 `addEventListener` 及 `removeEventListener` 一样！
举一个例子，我们在[【30 天精通 RxJS (04)： 什么是 Observable ?】](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(04)%EF%BC%9A%20%E4%BB%80%E4%B9%88%E6%98%AF%20Observable.md)实例的 Observer Pattern 就是类事件，代码如下：

```
class Producer {
	constructor() {
		this.listeners = [];
	}
	addListener(listener) {
		if(typeof listener === 'function') {
			this.listeners.push(listener)
		} else {
			throw new Error('listener 必须是 function')
		}
	}
	removeListener(listener) {
		this.listeners.splice(this.listeners.indexOf(listener), 1)
	}
	notify(message) {
		this.listeners.forEach(listener => {
			listener(message);
		})
	}
}
// ------- 以上都是之前的代码 -------- //

var egghead = new Producer(); 
// egghead 同时具有 注册观察者及移除观察者 两种方法

var source = Rx.Observable
    .fromEventPattern(
        (handler) => egghead.addListener(handler), 
        (handler) => egghead.removeListener(handler)
    );

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
})

egghead.notify('Hello! Can you hear me?');
// Hello! Can you hear me?

```

[JSBin](https://jsbin.com/wiruxej/1/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/xpkxhhz3/)

上面的代码可以看到，`egghead` 是 `Producer` 的实例，同时具有 注册监听及移除监听两种方法，我们可以将这两个方法依序传入 `fromEventPattern` 来建立 Observable 的物件实例！

> 
> 
> 这里要注意不要直接将方法传入，避免 this 出错！也可以用 `bind` 来写。
> 
> 

```
Rx.Observable
    .fromEventPattern(
        egghead.addListener.bind(egghead), 
        egghead.removeListener.bind(egghead)
    )
    .subscribe(console.log)

```

### empty, never, throw

接下来我们要看几个比较无趣的 operators，之后我们会讲到很多 observables 合并(combine)、转换(transforme)的方法，到那个时候无趣的 observable 也会很有用！

有点像是数学上的 **零(0)**，虽然有时候好像没什么，但却非常的重要。在 Observable 的世界里也有类似的东西，像是`empty`

```
var source = Rx.Observable.empty();

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});
// complete!

```

[JSBin](https://jsbin.com/xerori/7/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/5/)

`empty` 会给我们一个**空**的 observable，如果我们订阅这个 observable 会发生什么事呢？ 它会立即送出 complete 的讯息！

> 
> 
> 可以直接把 `empty` 想成没有做任何事，但它至少会告诉你它没做任何事。
> 
> 

数学上还有一个跟零(0)很像的数，那就是 **无穷(∞)**，在 Observable 的世界里我们用 `never` 来建立无穷的 observable

```
var source = Rx.Observable.never();

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

```

[JSBin](https://jsbin.com/xerori/9/edit?js,console,output) |[JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/6/)

never 会给我们一个无穷的 observable，如果我们订阅它又会发生什么事呢？...什么事都不会发生，它就是一个一直存在但却什么都不做的 observable。

> 
> 
> 可以把 never 想像成一个结束在无穷久以后的 observable，但你永远等不到那一天！
> 
> 

> 
> 
> 题外话，笔者一直很喜欢平行线的解释： 两条平行线就是它们相交于无穷远
> 
> 

最后还有一个 operator `throw`，它也就只做一件事就是抛出错误。

```
var source = Rx.Observable.throw('Oop!');

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// Throw Error: Oop!

```

[JSBin](https://jsbin.com/xerori/10/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/8/)

上面这段代码就只会 log 出 `'Throw Error: Oop!'`。

这三个 operators 虽然目前看起来没什么用，但之后在文章中大家就会慢慢发掘它们的用处！

### interval, timer

接着我们要看两个跟时间有关的 operators，在 JS 中我们可以用 `setInterval` 来建立一个持续的行为，这也能用在 Observable 中

```
var source = Rx.Observable.create(function(observer) {
    var i = 0;
    setInterval(() => {
        observer.next(i++);
    }, 1000)
});

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// 1
// 2
// .....

```

[JSBin](https://jsbin.com/xerori/11/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/9/)

上面这段代码，会每隔一秒送出一个从零开始递增的整数，在 Observable 的世界也有一个 operator 可以更方便地做到这件事，就是 `interval`

```
var source = Rx.Observable.interval(1000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// 1
// 2
// ...

```

[JSBin](https://jsbin.com/xerori/12/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/10/)

`interval` 有一个参数必须是数值(Number)，这的数值代表发出讯号的间隔时间(ms)。这两段代码基本上是等价的，会持续每隔一秒送出一个从零开始递增的数值！

另外有一个很相似的 operator 叫 `timer`， `timer` 可以给两个参数，范例如下

```
var source = Rx.Observable.timer(1000, 5000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// 1
// 2 ...

```

[JSBin](https://jsbin.com/xerori/13/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/11/)

当 `timer` 有两个参数时，第一个参数代表要发出第一个值的等待时间(ms)，第二个参数代表第一次之后发送值的间隔时间，所以上面这段代码会先等一秒送出 1 之后每五秒送出 2, 3, 4, 5...。

`timer` 第一个参数除了可以是数值(Number)之外，也可以是日期(Date)，就会等到指定的时间在发送第一个值。

另外 `timer` 也可以只接收一个参数

```
var source = Rx.Observable.timer(1000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// complete!

```

[JSBin](https://jsbin.com/xerori/14/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/e7u6k1b5/12/)

上面这段代码就会等一秒后送出 1 同时通知结束。

## Subscription

今天我们讲到很多 无穷的 observable，例如 interval, never。但有时我们可能会在某些行为后不需要这些资源，要做到这件事最简单的方式就是 `unsubscribe`。

其实在订阅 observable 后，会回传一个 subscription 物件，这个物件具有释放资源的`unsubscribe` 方法，范例如下

```
var source = Rx.Observable.timer(1000, 1000);

// 取得 subscription
var subscription = source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});

setTimeout(() => {
    subscription.unsubscribe() // 停止订阅(退订)， RxJS 4.x 以前的版本用 dispose()
}, 5000);
// 0
// 1
// 2
// 3
// 4

```

[JSBin](https://jsbin.com/xerori/15/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/d95a8peo/6/)

这里我们用了 `setTimeout` 在 5 秒后，执行了 `subscription.unsubscribe()` 来停止订阅并释放资源。另外 subscription 物件还有其他合并订阅等作用，这个我们之后有机会会在提到！

> 
> 
> Events observable 尽量不要用 `unsubscribe` ，通常我们会使用 `takeUntil`，在某个事件发生后来完成 Event observable，这个部份我们之后会讲到！
> 
> 

## 今日小结

今天我们把建立 Observable 实例的方法几乎都讲完了，建立 Observable 是 RxJS 的基础，接下来我们会讲转换(Transformation)、过滤(Filter)、合并(Combination)等 Operators，但不会像今天这样一次把一整个类型的 operator 讲完，笔者会依照实用程度以及范例搭配穿插著讲各种 operator!