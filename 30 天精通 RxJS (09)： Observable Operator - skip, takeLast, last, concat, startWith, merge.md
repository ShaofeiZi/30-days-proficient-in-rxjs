# 30 天精通 RxJS (09)： Observable Operator - skip, takeLast, last, concat, startWith, merge

这是【30天精通 RxJS】的 09 篇，如果还没看过 08 篇可以往这边走：
[30 天精通 RxJS (08)：简易拖拽实例 - take, first, takeUntil, concatAll](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(08)%EF%BC%9A%E7%AE%80%E6%98%93%E6%8B%96%E6%8B%BD%E5%AE%9E%E4%BE%8B%20-%20take%2C%20first%2C%20takeUntil%2C%20concatAll.md)

今天是美好的圣诞节，先祝读者们圣诞快乐！
为了让大家在圣诞节好好的陪家人，所以今天的文章内容就轻松点，让我们简单介绍几个的 operators 就好了。

## Operators

### skip

我们昨天介绍了 `take` 可以取前几个发送的元素，今天介绍可以略过前几个发送元素的 operator: `skip`，示例如下：

```
var source = Rx.Observable.interval(1000);
var example = source.skip(3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 3
// 4
// 5...

```

[JSBin](https://jsbin.com/rucopex/1/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/tucvcrmh/)

原本从 0 开始的就会变成从 3 开始，但是记得原本元素的等待时间仍然存在，也就是说此示例第一个取得的元素需要等 4 秒，用 Marble Diagram 表示如下。

```
source : ----0----1----2----3----4----5--....
                    skip(3)
example: -------------------3----4----5--...

```

### takeLast

除了可以用 take 取前几个之外，我们也可以倒过来取最后几个，示例如下：

```
var source = Rx.Observable.interval(1000).take(6);
var example = source.takeLast(2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 4
// 5
// complete

```

这里我们先取了前 6 个元素，再取最后两个。所以最后会发送 4, 5, complete，这里有一个重点，就是 takeLast 必须等到整个 observable 完成(complete)，才能知道最后的元素有哪些，并且**同步发送**，如果用 Marble Diagram 表示如下

```
source : ----0----1----2----3----4----5|
                takeLast(2)
example: ------------------------------(45)|

```

这里可以看到 takeLast 后，比须等到原本的 observable 完成后，才立即同步发送 4, 5, complete。

### last

跟 `take(1)` 相同，我们有一个 `takeLast(1)` 的简化写法，那就是 `last()` 用来取得最后一个元素。

```
var source = Rx.Observable.interval(1000).take(6);
var example = source.last();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 5
// complete

```

用 Marble Diagram 表示如下

```
source : ----0----1----2----3----4----5|
                    last()
example: ------------------------------(5)|

```

### concat

`concat` 可以把多个 observable 实例合并成一个，示例如下

```
var source = Rx.Observable.interval(1000).take(3);
var source2 = Rx.Observable.of(3)
var source3 = Rx.Observable.of(4,5,6)
var example = source.concat(source2, source3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4
// 5
// 6
// complete

```

[JSBin](https://jsbin.com/rucopex/2/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/tucvcrmh/1/)

跟 `concatAll` 一样，必须先等前一个 observable 完成(complete)，才会继续下一个，用 Marble Diagram 表示如下。

```
source : ----0----1----2|
source2: (3)|
source3: (456)|
            concat()
example: ----0----1----2(3456)|

```

另外 concat 还可以当作静态方法使用

```
var source = Rx.Observable.interval(1000).take(3);
var source2 = Rx.Observable.of(3);
var source3 = Rx.Observable.of(4,5,6);
var example = Rx.Observable.concat(source, source2, source3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/rucopex/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/tucvcrmh/2/)

### startWith

`startWith` 可以在 observable 的一开始塞要发送的元素，有点像 `concat` 但参数不是 observable 而是要发送的元素，使用示例如下

```
var source = Rx.Observable.interval(1000);
var example = source.startWith(0);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 0
// 1
// 2
// 3...

```

这里可以看到我们在 source 的一开始塞了一个 `0`，让 example 会在一开始就立即发送 `0`，用 Marble Diagram 表示如下

```
source : ----0----1----2----3--...
                startWith(0)
example: (0)----0----1----2----3--...

```

记得 startWith 的值是一开始就同步发出的，这个 operator 很常被用来保存程序的起始状态！

### merge

`merge` 跟 `concat` 一样都是用来合并 observable，但他们在行为上有非常大的不同！

让我们直接来看例子吧

```
var source = Rx.Observable.interval(500).take(3);
var source2 = Rx.Observable.interval(300).take(6);
var example = source.merge(source2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 0
// 1
// 2
// 1
// 3
// 2
// 4
// 5
// complete

```

[JSBin](https://jsbin.com/rucopex/6/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/tucvcrmh/3/)

上面可以看得出来，`merge` 把多个 observable 同时处理，这跟 `concat` 一次处理一个 observable 是完全不一样的，由于是同时处理行为会变得较为复杂，这里我们用 Marble Diagram 会比较好解释。

```
source : ----0----1----2|
source2: --0--1--2--3--4--5|
            merge()
example: --0-01--21-3--(24)--5|

```

这里可以看到 `merge` 之后的 example 在时间序上同时在跑 source 与 source2，当两件事情同时发生时，会同步发送资料(被 merge 的在后面)，当两个 observable 都结束时才会真的结束。

merge 同样可以当作静态方法用

```
var source = Rx.Observable.interval(500).take(3);
var source2 = Rx.Observable.interval(300).take(6);
var example = Rx.Observable.merge(source, source2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

merge 的逻辑有点像是 OR(||)，就是当两个 observable 其中一个被触发时都可以被处理，这很常用在一个以上的按钮具有部分相同的行为。

例如一个影片播放器有两个按钮，一个是暂停(II)，另一个是结束播放(口)。这两个按钮都具有相同的行为就是影片会被停止，只是结束播放会让影片回到 00 秒，这时我们就可以把这两个按钮的事件 merge 起来处理影片暂停这件事。

```
var stopVideo = Rx.Observable.merge(stopButton, endButton);

stopVideo.subscribe(() => {
    // 暂停播放影片
})

```

## 今日小结

今天介绍的六个 operators 都是平时很容易用到的，我们之后的示例也有机会再遇到。希望读者们能自己试试这些方法，之后使用时会比较有印象！

不知道读者今天有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，谢谢。