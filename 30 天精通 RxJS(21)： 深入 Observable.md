# 30 天精通 RxJS(21): 深入 Observable

我们已经把绝大部分的 operators 都介绍完了，但一直没有机会好好的解释 Observable 的 operators 运行方式。在系列文章的一开头是以数组(Array)的 operators(map, filter, concatAll) 作为切入点，让读者们在学习 observable 时会更容易接受跟理解，但实际上 observable 的 operators 跟数组的有很大的不同，主要差异有两点

1.  延迟运算
2.  渐进式取值

## 延迟运算

延迟运算很好理解，所有 Observable 一定会等到订阅后才开始对元素做运算，如果没有订阅就不会有运算的行为

```javascript
var source = Rx.Observable.from([1,2,3,4,5]);
var example = source.map(x => x + 1);

```

上面这段代码因为 Observable 还没有被订阅，所以不会真的对元素做运算，这跟数组的操作不一样，如下

```javascript
var source = [1,2,3,4,5];
var example = source.map(x => x + 1); 

```

上面这段代码执行完，example 就已经取得所有元素的返回值了。

延迟运算是 Observable 跟数组最明显的不同，延迟运算所带来的优势在之前的文章也已经提过这里就不再赘述，因为我们还有一个更重要的差异要讲，那就是**渐进式取值**

## 渐进式取值

数组的 operators 都必须完整的运算出每个元素的返回值并组成一个数组，再做下一个 operator 的运算，我们看下面这段代码

```javascript
var source = [1,2,3];
var example = source
              .filter(x => x % 2 === 0) // 这里会运算并返回一个完整的数组
              .map(x => x + 1) // 这里也会运算并返回一个完整的数组

```

上面这段代码，相信读者们都很熟悉了，大家应该都有注意到 `source.filter(...)` 就会返回一整个新数组，再接下一个 operator 又会再返回一个新的数组，这一点其实在我们实例 map 跟 filter 时就能观察到

```javascript
Array.prototype.map = function(callback) {
    var result = []; // 建立新数组
    this.forEach(function(item, index, array) {
        result.push(callback(item, index, array))
    });
    return result; // 返回新数组
}

```

每一次的 operator 的运算都会建立一个新的数组，并在每个元素都运算完后返回这个新数组，我们可以用下面这张动态图表示运算过程

![](http://i.giphy.com/l0HlPZeB9OvFu7QwE.gif)

Observable operator 的运算方式跟数组的是完全的不同，虽然 Observable 的 operator 也都会回传一个新的 observable，但因为元素是渐进式取得的关系，所以每次的运算是一个元素运算到底，而不是运算完全部的元素再返回。

```javascript
var source = Rx.Observable.from([1,2,3]);
var example = source
              .filter(x => x % 2 === 0)
              .map(x => x + 1)

example.subscribe(console.log);

```

上面这段代码运行的方式是这样的

1.  送出 `1` 到 filter 被过滤掉
2.  送出 `2` 到 filter 在被送到 map 转成 `3`，送到 observer `console.log` 印出
3.  送出 `3` 到 filter 被过滤掉

每个元素送出后就是运算到底，在这个过程中不会等待其他的元素运算。这就是渐进式取值的特性，不知道读者们还记不记得我们在讲 Iterator 跟 Observer 时，就特别强调这两个 Pattern 的共同特性是渐进式取值，而我们在实例 Iterator 的过程中其实就能看出这个特性的运行方式

```javascript
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

var myIterator = new IteratorFromArray([1,2,3]);
myIterator.map(x => x + 1);
myIterator.next(); // { done: false, value: 2 }

```

虽然上面这段代码是一个非常简单的示范，但可以看得出来每一次 map 虽然都会返回一个新的 oterator，但实际上在做元素运算时，因为渐进式的特性会使一个元素运算到底，Observable 也是相同的概念，我们可以用下面这张动态图表示运算过程

![](http://i.giphy.com/3o6ZtqrBfUyHvMDQ2c.gif)

渐进式取值的观念在 Observable 中其实非常的重要，这个特性也使得 Observable 相较于 Array 的 operator 在做运算时来的高效很多，尤其是在处理大量资料的时候会非常明显！

## 今日小结

今天我们讲解了 Observable 跟数组各自 operators 运行上的差异，这些细微的差异实际上对程式的运行效率有着很大的影响。

从我们一开始从数组作为 obsevable 的切入点，中间介绍了各种常用的 operator，到今天我们厘清了数组跟 Observable 运行上的差异，在 Observable 这块我们几乎已经完成了，剩下的是一些衍生出来的东西，像是 multicast, publish... 等，这些我们会在介绍完 Subject 后在做说明！