# 30 天精通 RxJS (05)： 建立 Observable(一)

> 
> 
> Observable 是 RxJS 的核心，今天让我们从如何建立 Observable 开始！
> 
> 

这是【30天精通 RxJS】的 05 篇，如果还没看过 04 篇可以往这边走：
[30 天精通 RxJS (04)： 什么是 Observable ?](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(04)%EF%BC%9A%20%E4%BB%80%E4%B9%88%E6%98%AF%20Observable.md)

不想看文章的人，可以直接看视频喔！

> 
> 
> 今天大家看文章一定要分清楚 **Observable** 跟 **Observer**，不要搞混。
> 
> 

前几天我们把所有重要的观念及前置的知识都讲完了，今天要正式进入 RxJS 的应用，整个 RxJS 说白了就是**一个核心三个重点**。

一个核心是 Observable 再加上相关的 Operators(map, filter...)，这个部份是最重要的，其他三个重点本质上也是围绕着这个核心在转，所以我们会花将近 20 天的篇数讲这个部份的观念及使用案例。

另外三个重点分别是

*   Observer(观察者)
*   Subject(订阅者)
*   Schedulers(操作符)

Observer 是这三个当中一定会用到却是最简单的，所以我们今天就会把它介绍完。Subject 一般应用到的频率就相对低很多，但如果想要看懂 RxJS 相关的 Library 或 Framework，Subject 就是一定要会的重点，所以这个部份我们大概会花 3-5 天的时间讲解。至于 Schedulers 则是要解决 RxJS 衍伸出的最后一道问题，这个部份会视情况加入或是在 30 天后补完。

![redux-observable logo](https://redux-observable.js.org/logo/logo-small.gif)

> 
> 
> [redux-observable](https://github.com/redux-observable/redux-observable) 就是用了 Subject 实例的
> 
> 

> 
> 
> 让我卖个关子，先不说 RxJS 最后一道问题是什么。
> 
> 

说了这么多，我们赶快进入到今天的主题 Observable 吧！

## 建立 Observable: `create`

建立 Observable 的方法有非常多种，其中 `create` 是最基本的方法。`create` 方法在 `Rx.Observable` 物件中，要传入一个 callback function ，这个 callback function 会接收一个 observer 参数，如下

```
var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); // RxJS 4.x 以前的版本用 onNext
		observer.next('Anna');
	})

```

这个 callback function 会定义 observable 将会如何发送值。

> 
> 
> 虽然 Observable 可以被 `create`，但实际上我们通常都使用 **creation operator** 像是 from, of, fromEvent, fromPromise 等。这里只是为了从基本的开始讲解所以才用 `create`
> 
> 

我们可以订阅这个 observable，来接收他发送的值，代码如下

```
var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); // RxJS 4.x 以前的版本用 onNext
		observer.next('Anna');
	})

// 订阅这个 observable	
observable.subscribe(function(value) {
	console.log(value);
})

```

[JSBin](https://jsbin.com/vetoti/1/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/yL8n4v53/)

当我们订阅这个 observable，他就会依序发送 `'Jerry'` `'Anna'` 两个字串。

> 
> 
> 订阅 Observable 跟 addEventListener 在实例上其实有非常大的不同。虽然在行为上很像，但实际上 Observable 根本没有管理一个订阅的清单，这个部份的细节我们留到最后说明！
> 
> 

这里有一个重点，很多人认为 RxJS 是在做非同步处理，所以所有行为都是非同步的。但其实这个观念是错的，RxJS 确实主要在处理非同步行为没错，但也同时能处理同步行为，像是上面的代码就是同步执行的。

证明如下

```
var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); // RxJS 4.x 以前的版本用 onNext
		observer.next('Anna');
	})

console.log('start');
observable.subscribe(function(value) {
	console.log(value);
});
console.log('end');

```

[JSBin](https://jsbin.com/vetoti/2/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/yL8n4v53/1/)

上面这段代码会印出

```
start
Jerry
Anna
end

```

而不是

```
start
end
Jerry
Anna

```

所以很明显的这段代码是同步执行的，当然我们可以拿它来处理非同步的行为！

```
var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); // RxJS 4.x 以前的版本用 onNext
		observer.next('Anna');

		setTimeout(() => {
			observer.next('RxJS 30 days!');
		}, 30)
	})

console.log('start');
observable.subscribe(function(value) {
	console.log(value);
});
console.log('end');

```

[JSBin](https://jsbin.com/vetoti/4/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/yL8n4v53/2/)

这时就会印出

```
start
Jerry
Anna
end
RxJS 30 days!

```

从上述的代码能看得出来

**Observable 同时可以处理同步与非同步的行为！**

## 观察者 Observer

Observable 可以被订阅(subscribe)，或说可以被观察，而订阅 Observable 的物件又称为 **观察者(Observer)**。观察者是一个具有三个方法(method)的物件，每当 Observable 发生事件时，便会呼叫观察者相对应的方法。

> 
> 
> 注意这里的观察者(Observer)跟上一篇讲的观察者模式(Observer Pattern)无关，观察者模式是一种设计模式，是思考问题的解决过程，而这里讲的观察者是一个被定义的物件。
> 
> 

观察者的三个方法(method)：

*   next：每当 Observable 发发送新的值，next 方法就会被呼叫。

*   complete：在 Observable 没有其他的资料可以取得时，complete 方法就会被呼叫，在 complete 被呼叫之后，next 方法就不会再起作用。

*   error：每当 Observable 内发生错误时，error 方法就会被呼叫。

说了这么多，我们还是直接来建立一个观察者吧！

```
var observable = Rx.Observable
	.create(function(observer) {
			observer.next('Jerry');
			observer.next('Anna');
			observer.complete();
			observer.next('not work');
	})

// 建立一个观察者，具备 next, error, complete 三个方法
var observer = {
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log(error)
	},
	complete: function() {
		console.log('complete')
	}
}

// 用我们定义好的观察者，来订阅这个 observable	
observable.subscribe(observer)

```

[JSBin](https://jsbin.com/vetoti/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/yL8n4v53/3/)

上面这段代码会印出

```
Jerry
Anna
complete

```

上面的示例可以看得出来在 complete 执行后，next 就会自动失效，所以没有印出 `not work`。

下面则是发送错误的示例

```
var observable = Rx.Observable
  .create(function(observer) {
    try {
      observer.next('Jerry');
      observer.next('Anna');
      throw 'some exception';
    } catch(e) {
      observer.error(e)
    }
  });

// 建立一个观察者，具备 next, error, complete 三个方法
var observer = {
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log('Error: ', error)
	},
	complete: function() {
		console.log('complete')
	}
}

// 用我们定义好的观察者，来订阅这个 observable	
observable.subscribe(observer)

```

[JSBin](https://jsbin.com/poyefom/1/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/kf6dphqp/)

这里就会执行 error 的 function 印出 `Error: some exception`。

另外观察者可以是不完整的，他可以只具有一个 next 方法，如下

```
var observer = {
	next: function(value) {
		//...
	}
}

```

> 
> 
> 有时候 Observable 会是一个无限的序列，例如 click 事件，这时 `complete` 方法就有可能永远不会被呼叫！
> 
> 

我们也可以直接把 next, error, complete 三个 function 依序传入 `observable.subscribe`，如下：

```
observable.subscribe(
    value => { console.log(value); },
    error => { console.log('Error: ', error); },
    () => { console.log('complete') }
)

```

`observable.subscribe` 会在内部自动组成 observer 物件来操作。

## 实例细节

我们前面提到了，其实 Observable 的订阅跟 addEventListener 在实例上有蛮大的差异，虽然他们的行为很像！

addEventListener 本质上就是 Observer Pattern 的实例，在内部会有一份订阅清单，像是我们昨天实例的 Producer

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

```

我们在内部储存了一份所有的观察者清单(`this.listeners`)，在要发布通知时会对逐一的呼叫这份清单的观察者。

但在 Observable 不是这样实例的，在其内部并没有一份订阅者的清单。订阅 Observable 的行为比较像是执行一个物件的方法，并把资料传进这个方法中。

我们以下面的代码做说明

```
var observable = Rx.Observable
	.create(function (observer) {
			observer.next('Jerry');
			observer.next('Anna');
	})

observable.subscribe({
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log(error)
	},
	complete: function() {
		console.log('complete')
	}
})

```

像上面这段程式，他的行为比较像这样

```

function subscribe(observer) {
		observer.next('Jerry');
		observer.next('Anna');
}

subscribe({
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log(error)
	},
	complete: function() {
		console.log('complete')
	}
});

```

这里可以看到 subscribe 是一个 function，这个 function 执行时会传入观察者，而我们在这个 function 内部去执行观察者的方法。

**订阅一个 Observable 就像是执行一个 function**

## 今日小结

今天在讲关于建立 Observable 的实例，用到了 `create` 的方法，但大部分的内容还是在讲 Observable 几个重要的观念，如下

*   Observable 可以同时处理**同步**跟**非同步**行为
*   Observer 是一个物件，这个物件具有三个方法，分别是 **next**, **error**, **complete**
*   订阅一个 Observable 就像在执行一个 function

不知道读者是否有所收获，如果有任何问题或建议，欢迎在下方留言给我，谢谢。