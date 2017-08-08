# 30 天精通 RxJS(17): Observable Operators - switch, mergeAll, concatAll

今天我们要讲三个 operators，这三个 operators 都是用来处理 Higher Order Observable。所谓的 Higher Order Observable 就是指一个 Observable 送出的元素还是一个 Observable，就像是二维数组一样，一个数组中的每个元素都是数组。如果用泛型来表达就像是

```
Observable<Observable<T>>

```

通常我们需要的是第二层 Observable 送出的元素，所以我们希望可以把二维的 Observable 改成一维的，像是下面这样

```
Observable<Observable<T>> => Observable<T>

```

其实想要做到这件事有三个方法 switch、mergeAll 和 concatAll，其中 concatAll 我们在之前的文章已经稍微讲过了，今天这篇文章会讲解这三个 operators 各自的效果跟差异。

## Operators

### concatAll

我们在讲简易拖拽的示例时就有讲过这个 operator，concatAll 最重要的重点就是他会处理完前一个 observable 才会在处理下一个 observable，让我们来看一个示例

```javascript
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// (点击后)
// 0
// 1
// 2
// 3
// 4
// 5 ...

```

[JSBin](https://jsbin.com/numuji/2/edit?js,console,output)

上面这段代码，当我们点击画面时就会开始送出数值，如果用 Marble Diagram 表示如下

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     concatAll()
example: ----------------0----1----2----3----4--..

```

从 Marble Diagram 可以看得出来，当我们点击一下 click 事件会被转成一个 observable 而这个 observable 会每一秒送出一个递增的数值，当我们用 concatAll 之后会把二维的 observable 摊平成一维的 observable，但 **concatAll 会一个一个处理，一定是等前一个 observable 完成(complete)才会处理下一个 observable**，因为现在送出 observable 是无限的永远不会完成(complete)，就导致他永远不会处理第二个送出的 observable!

我们再看一个例子

```javascript
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000).take(3));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

现在我们把送出的 observable 限制只取前三个元素，用 Marble Diagram 表示如下

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \
                    \ ----0----1----2|   ----0----1----2|
                     ----0----1----2|
                     concatAll()
example: ----------------0----1----2----0----1----2--..

```

这里我们把送出的 observable 变成有限的，只会送出三个元素，这时就能看得出来 concatAll 不管两个 observable 送出的时间多么相近，一定会先处理前一个 observable 再处理下一个。

### switch

switch 同样能把二维的 observable 摊平成一维的，但他们在行为上有很大的不同，我们来看下面这个示例

```javascript
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.switch();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/numuji/edit?js,console,output)

用 Marble Diagram 表示如下

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \----0----1--...
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     switch()
example: -----------------0----1----2--------0----1--...

```

switch 最重要的就是他会**在新的 observable 送出后直接处理新的 observable 不管前一个 observable 是否完成，每当有新的 observable 送出就会直接把旧的 observable 退订(unsubscribe)，永远只处理最新的 observable!**

所以在这上面的 Marble Diagram 可以看得出来第一次送出的 observable 跟第二次送出的 observable 时间点太相近，导致第一个 observable 还来不及送出元素就直接被退订了，当下一次送出 observable 就又会把前一次的 observable 退订。

### mergeAll

我们之前讲过 merge 他可以让多个 observable 同时送出元素，mergeAll 也是同样的道理，它会把二维的 observable 转成一维的，并且能够同时处理所有的 observable，让我们来看这个示例

```javascript
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.mergeAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

上面这段代码用 Marble Diagram 表示如下

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \----0----1--...
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     switch()
example: ----------------00---11---22---33---(04)4--...

```

从 Marble Diagram 可以看出来，所有的 observable 是并行(Parallel)处理的，也就是说 mergeAll 不会像 switch 一样退订(unsubscribe)原先的 observable 而是并行处理多个 observable。以我们的示例来说，当我们点击越多下，最后送出的频率就会越快。

另外 mergeAll 可以传入一个数值，这个数值代表他可以同时处理的 observable 数量，我们来看一个例子

```javascript
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000).take(3));

var example = source.mergeAll(2);
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

这里我们送出的 observable 改成取前三个，并且让 mergeAll 最多只能同时处理 2 个 observable，用 Marble Diagram 表示如下

```
click  : ---------c-c----------o----------.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o----------c----------..
                   \ \          \----0----1----2|     
                    \ ----0----1----2|  
                     ----0----1----2|
                     mergeAll(2)
example: ----------------00---11---22---0----1----2--..

```

当 mergeAll 传入参数后，就会等处理中的其中一个 observable 完成，再去处理下一个。以我们的例子来说，前面两个 observabel 可以被并行处理，但第三个 observable 必须等到第一个 observable 结束后，才会开始。

我们可以利用这个参数来决定要同时处理几个 observable，如果我们传入 `1` 其行为就会跟 `concatAll` 是一模一样的，这点在原始码可以看到他们是完全相同的。

## 今日小结

今天介绍了三个可以处理 High Order Observable 的方法，并讲解了三个方法的差异，不知道读者有没有收获呢？ 如果有任何问题欢迎在下方留言给我，感谢！