# 30 天精通 RxJS (04)： 什么是 Observable ?

> 
> 
> 整个 RxJS 的基础就是 Observable，只要弄懂 Observable 就算是学会一半的 RxJS 了，剩下的就只是一些方法的练习跟熟悉；但到底什么是 Observable 呢？
> 
> 

这是【30天精通 RxJS】的 04 篇，如果还没看过 03 篇可以往这边走：
[30 天精通 RxJS (03)： Functional Programming 通用函数](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(03)%EF%BC%9A%20Functional%20Programming%20%E9%80%9A%E7%94%A8%E5%87%BD%E6%95%B0.md)

要理解 Observable 之前，我们必须先谈谈两个设计模式(Design Pattern)， Iterator Pattern 跟 Observer Pattern。今天这篇文章会带大家快速的了解这两个设计模式，并解释这两个 Pattern 跟 Observable 之间的关系！

## Observer Pattern(观察者模式)

Observer Pattern 其实很常遇到，在许多 API 的设计上都用了 Observer Pattern 实例，最简单的例子就是 DOM 事件的事件监听，代码如下

```
function clickHandler(event) {
	console.log('user click!');
}

document.body.addEventListener('click', clickHandler)

```

在上面的代码，我们先建立了一个 `clickHandler`函数，再用 DOM 事件 (示例是 body) 的 `addEventListener` 来监听**点击**(click)事件，每次使用者在 body 点击滑鼠就会执行一次 `clickHandler`，并把相关的资讯(event)带进来！这就是观察者模式，我们可以对某件事注册监听，并在事件发生时，自动执行我们注册的观察者(listener)。

Observer 的观念其实就这么的简单，但笔者希望能透过代码带大家了解，如何实例这样的 Pattern！

首先我们需要一个建构式，这个建构式 new 出来的实例可以被监听。

> 
> 
> 这里我们先用 ES5 的写法，会再附上 ES6 的写法
> 
> 

```
function Producer() {

	// 这个 if 只是避免使用者不小心把 Producer 当作函数来调用
	if(!(this instanceof Producer)) {
	  throw new Error('请用 new Producer()!');
	  // 仿 ES6 行为可用： throw new Error('Class constructor Producer cannot be invoked without 'new'')
	}

	this.listeners = [];
}

// 加入监听的方法
Producer.prototype.addListener = function(listener) {
	if(typeof listener === 'function') {
		this.listeners.push(listener)
	} else {
		throw new Error('listener 必须是 function')
	}
}

// 移除监听的方法
Producer.prototype.removeListener = function(listener) {
	this.listeners.splice(this.listeners.indexOf(listener), 1)
}

// 发送通知的方法
Producer.prototype.notify = function(message) {
	this.listeners.forEach(listener => {
		listener(message);
	})
}

```

> 
> 
> 这里用到了 this, prototype 等观念，大家不了解可以去看我的一支[视频](https://youtu.be/BlT6pCG2M1I)专门讲解这几个观念！
> 
> 

附上 ES6 版本的代码，跟上面代码的行为基本上是一样的

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

有了上面的代码后，我们就可以来建立事件实例了

```
var egghead = new Producer(); 
// new 出一个 Producer 实例叫 egghead

function listener1(message) {
	console.log(message + 'from listener1');
}

function listener2(message) {
	console.log(message + 'from listener2');
}

egghead.addListener(listener1); // 注册监听
egghead.addListener(listener2);

egghead.notify('A new course!!') // 当某件事情方法时，执行

```

当我们执行到这里时，会印出：

```
a new course!! from listener1
a new course!! from listener2

```

每当 `egghead.notify` 执行一次，`listener1` 跟 `listener2` 就会被通知，而这些 listener 可以额外被添加，也可以被移除！

虽然我们的实例很简单，但它很好的说明了 Observer Pattern 如何在**事件**(event)跟**观察者**(listener)的互动中做到去耦合(decoupling)。

## Iterator（迭代器） Pattern

Iterator 是一个事件，它的就像是一个指针(pointer)，指向一个资料结构并产生一个序列(sequence)，这个序列会有资料结构中的所有元素(element)。

先让我们来看看原生的 JS 要怎么建立 iterator

```
var arr = [1, 2, 3];

var iterator = arr[Symbol.iterator]();

iterator.next();
// { value: 1, done: false }
iterator.next();
// { value: 2, done: false }
iterator.next();
// { value: 3, done: false }
iterator.next();
// { value: undefined, done: true }

```

> 
> 
> JavaScript 到了 ES6 才有原生的 Iterator
> 
> 

> 
> 
> 在 ECMAScript 中 Iterator 最早其实是要采用类似 Python 的 Iterator 规范，就是 Iterator 在没有元素之后，执行 `next` 会直接抛出错误；但后来经过一段时间讨论后，决定采更 functional 的做法，改成在取得最后一个元素之后执行 `next` 永远都回传 `{ done: true, value: undefined }`
> 
> 

JavaScript 的 Iterator 只有一个 next 方法，这个 next 方法只会回传这两种结果：

1.  在最后一个元素前： `{ done: false, value: elem }`
2.  在最后一个元素之后： `{ done: true, value: undefined }`

当然我们可以自己实例简单的 Iterator Pattern

```
function IteratorFromArray(arr) {
	if(!(this instanceof IteratorFromArray)) {
		throw new Error('请用 new IteratorFromArray()!');
	}
	this._array = arr;
	this._cursor = 0;	
}

IteratorFromArray.prototype.next = function() {
	return this._cursor < this._array.length ?
		{ value: this._array[this._cursor++], done: false } :
		{ done: true };
}

```

附上 ES6 版本的代码，行为同上

```
class IteratorFromArray {
	constructor(arr) {
		this._array = arr;
		this._cursor = 0;
	}

	next() {
		return this._cursor < this._array.length ?
		{ value: this._array[this._cursor++], done: false } :
		{ done: true };
	}
}

```

Iterator Pattern 虽然很单纯，但同时带来了两个优势，第一它渐进式取得资料的特性可以拿来做延迟运算(Lazy evaluation)，让我们能用它来处理大资料结构。第二因为 iterator 本身是序列，所以可以实例所有数组的运算方法像 map, filter... 等！

这里我们利用最后一段代码实例 map 试试

```
class IteratorFromArray {
	constructor(arr) {
		this._array = arr;
		this._cursor = 0;
	}

	next() {
		return this._cursor < this._array.length ?
		{ value: this._array[this._cursor++], done: false } :
		{ done: true };
	}

	map(callback) {
		const iterator = new IteratorFromArray(this._array);
		return {
			next: () => {
				const { done, value } = iterator.next();
				return {
					done: done,
					value: done ? undefined : callback(value)
				}
			}
		}
	}
}

var iterator = new IteratorFromArray([1,2,3]);
var newIterator = iterator.map(value => value + 3);

newIterator.next();
// { value: 4, done: false }
newIterator.next();
// { value: 5, done: false }
newIterator.next();
// { value: 6, done: false }

```

### 补充: 延迟运算(Lazy evaluation)

延迟运算，或说 call-by-need，是一种运算策略(evaluation strategy)，简单来说我们延迟一个表达式的运算时机直到真正需要它的值在做运算。

以下我们用 generator（生成器） 实例 iterator 来举一个例子

```
	function* getNumbers(words) {
		for (let word of words) {
			if (/^[0-9]+$/.test(word)) {
			    yield parseInt(word, 10);
			}
		}
	}

	const iterator = getNumbers('30 天精通 RxJS (04)');

	iterator.next();
	// { value: 3, done: false }
	iterator.next();
	// { value: 0, done: false }
	iterator.next();
	// { value: 0, done: false }
	iterator.next();
	// { value: 4, done: false }
	iterator.next();
	// { value: undefined, done: true }

```

这里我们写了一个函数用来抓取字串中的数字，在这个函数中我们用 for...of 的方式来取得每个字元并用正则表示式来判断是不是数值，如果为真就转成数值并回传。当我们把一个字串丢进 `getNumbers`函数时，并没有马上运算出字串中的所有数字，必须等到我们执行 `next()` 时，才会真的做运算，这就是所谓的延迟运算(evaluation strategy)

## Observable

在了解 Observer 跟 Iterator 后，不知道大家有没有发现其实 Observer 跟 Iterator 有个共通的特性，就是他们都是 **渐进式**(progressive) 的取得资料，差别只在于 Observer 是生产者(Producer)推送资料(push)，而 Iterator 是消费者(Consumer)要求资料(pull)!

![push & pull](https://res.cloudinary.com/dohtkyi84/image/upload/v1482240798/push_pull.png)

Observable 其实就是这两个 Pattern（模式） 思想的结合，Observable 具备**生产者推送资料**的特性，同时能像序列，拥有**序列处理资料的方法**(map, filter...)！

更简单的来说，**Observable 就像是一个序列，里面的元素会随着时间推送**。

> 
> 
> 注意这里讲的是 **思想的结合**，Observable 跟 Observer 在实例上还是有差异，这我们在下一篇文章中讲到。
> 
> 

## 今日小结

今天讲了 Iterator 跟 Observer 两个 Pattern，这两个 Pattern 都是渐进式的取得元素，差异在于 Observer 是靠生产者推送资料，Iterator 则是消费者去要求资料，而 Observable 就是这两个思想的结合！

今天的观念需要比较多的思考，希望读者能多花点耐心想一想，如果有任何问题请在下方留言给我。