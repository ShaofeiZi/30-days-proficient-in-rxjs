# 30 天精通 RxJS(15): Observable Operators - distinct, distinctUntilChanged

新的一年马上就要到了，各位读者都去哪里跨年呢？ 笔者很可怜的只能一边写文章一边跨年，今天就简单看几个 operators 让大家好好跨年吧！

昨天我们讲到了 throttle 跟 debounce 两个方法来做性能优化，其实还有另一个方法可以做性能的优化处理，那就是 distinct。

## Operators

### distinct

如果会下 SQL 指令的应该都对 distinct 不陌生，它能帮我们把相同值的资料滤掉只留一笔，RxJS 里的 distinct 也是相同的作用，让我们直接来看范例

```javascript
var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'b'])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var example = source.distinct()

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// a
// b
// c
// complete

```

[JSBin](https://jsbin.com/dipabe/2/edit?js,console) | [JSFiddle](https://jsfiddle.net/3pfs88g8/)

如果用 Marble Diagram 表示如下

```
source : --a--b--c--a--b|
            distinct()
example: --a--b--c------|

```

从上面的范例可以看得出来，当我们用 distinct 后，只要有重复出现的值就会被过滤掉。

另外我们可以传入一个 selector callback function，这个 callback function 会传入一个接收到的元素，并回传我们真正希望比对的值，举例如下

```javascript
var source = Rx.Observable.from([{ value: 'a'}, { value: 'b' }, { value: 'c' }, { value: 'a' }, { value: 'c' }])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var example = source.distinct((x) => {
    return x.value
});

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// {value: "a"}
// {value: "b"}
// {value: "c"}
// complete

```

[JSBin](https://jsbin.com/dipabe/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/3pfs88g8/2/)

这里可以看到，因为 source 送出的都是实例，而 js 物件的比对是比对内存位置，所以在这个例子中这些实例永远不会相等，但实际上我们想比对的是实例中的 value，这时我们就可以传入 selector callback，来选择我们要比对的值。

> 
> 
> distinct 传入的 callback 在 RxJS 5 几个 bate 版本中有过很多改变，现在网路上很多文章跟教学都是过时的，请读者务必小心！
> 
> 

实际上 `distinct()` 会在背地里建立一个 Set，当接收到元素时会先去判断 Set 内是否有相同的值，如果有就不送出，如果没有则存到 Set 并送出。所以记得尽量不要直接把 distinct 用在一个无限的 observable 里，这样很可能会让 Set 越来越大，建议大家可以放第二个参数 flushes，或用 distinctUntilChanged

> 
> 
> 这里指的 Set 其实是 RxJS 自己实现的，跟 ES6 原生的 Set 行为也都一致，只是因为 ES6 的 Set 支持程度还并不理想，所以这里是直接用 JS 实现。
> 
> 

distinct 可以传入第二个参数 flushes observable 用来清除暂存的资料，范例如下

```javascript
var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'c'])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var flushes = Rx.Observable.interval(1300);
var example = source.distinct(null, flushes);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// a
// b
// c
// c
// complete

```

[JSBin](https://jsbin.com/dipabe/4/edit?js,console) | [JSFiddle](https://jsfiddle.net/3pfs88g8/3/)

这里我们用 Marble Diagram 比较好表示

```
source : --a--b--c--a--c|
flushes: ------------0---...
        distinct(null, flushes);
example: --a--b--c-----c|

```

其实 flushes observable 就是在送出元素时，会把 distinct 的暂存清空，所以之后的暂存就会从头来过，这样就不用担心暂存的 Set 越来愈大的问题，但其实我们平常不太会用这样的方式来处理，通常会用另一个方法 distinctUntilChanged。

### distinctUntilChanged

distinctUntilChanged 跟 distinct 一样会把相同的元素过滤掉，但 distinctUntilChanged 只会跟最后一次送出的元素比较，不会每个都比，举例如下

```
var source = Rx.Observable.from(['a', 'b', 'c', 'c', 'b'])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var example = source.distinctUntilChanged()

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// a
// b
// c
// b
// complete

```

[JSBin](https://jsbin.com/dipabe/6/edit?js,console) | [JSFiddle](https://jsfiddle.net/3pfs88g8/4/)

这里 distinctUntilChanged 只会暂存一个元素，并在收到元素时跟暂存的元素比对，如果一样就不送出，如果不一样就把暂存的元素换成刚接收到的新元素并送出。

```
source : --a--b--c--c--b|
            distinctUntilChanged()
example: --a--b--c-----b|

```

从 Marble Diagram 中可以看到，第二个 c 送出时刚好上一个就是 c 所以就被滤掉了，但最后一个 b 则跟上一个不同所以没被滤掉。

distinctUntilChanged 是比较常在开发中上使用的，最常见的状况是我们在做多方同步时。当我们有多个 Client，且每个 Client 有着各自的状态，Server 会再一个 Client 需要变动时通知所有 Client 更新，但可能某些 Client 接收到新的状态其实跟上一次收到的是相同的，这时我们就可用 distinctUntilChanged 方法只处理跟最后一次不相同的讯息，像是多方通话、多装置的资讯同步都会有类似的情境。

## 今日小结

今天讲了两个 distinct 方法，这两个方法平常可能用不太到，但在需求复杂的应用里是不可或缺的好方法，尤其要处理非常多人即时同步的情境下，这会是非常好用的方法，不知道读者们今天有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，感谢！