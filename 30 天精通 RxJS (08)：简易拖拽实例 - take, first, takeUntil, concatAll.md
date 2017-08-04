# 30 天精通 RxJS (08)：简易拖拽实例 - take, first, takeUntil, concatAll

> 
> 
> 如果是你会如何实现拖拽的功能？
> 
> 

这是【30天精通 RxJS】的 08 篇，如果还没看过 07 篇可以往这边走：
[30 天精通 RxJS (07)： Observable Operators & Marble Diagrams](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(07)%EF%BC%9A%20Observable%20Operators%20%26%20Marble%20Diagrams.md)

今天建议大家直接看[影片](https://www.youtube.com/watch?v=bgi3Uaab1ok )

我们今天要接着讲 take, first, takeUntil, concatAll 这四个 operators，并且实例一个简易的拖拽功能。

## Operators

### take

take 是一个很简单的 operator，顾名思义就是取前几个元素后就结束，示例如下

```
var source = Rx.Observable.interval(1000);
var example = source.take(3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// complete

```

[JSBin](https://jsbin.com/jogesut/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/ckyjuuva/1/)

这里可以看到我们的 `source` 原本是会发出无限个元素的，但这里我们用 `take(3)` 就会只取前 3 个元素，取完后就直接结束(complete)。

用 Marble diagram 表示如下

```
source : -----0-----1-----2-----3--..
                take(3)
example: -----0-----1-----2|

```

### first

first 会取 observable 发送的第 1 个元素之后就直接结束，行为跟 take(1) 一致。

```
var source = Rx.Observable.interval(1000);
var example = source.first();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// 0
// complete

```

[JSBin](https://jsbin.com/jogesut/5/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/ckyjuuva/3/)

用 Marble diagram 表示如下

```
source : -----0-----1-----2-----3--..
                first()
example: -----0|

```

### takeUntil

在实务上 takeUntil 很常使用到，他可以在某件事情发生时，让一个 observable 发送 完成(complete)讯息，示例如下

```
var source = Rx.Observable.interval(1000);
var click = Rx.Observable.fromEvent(document.body, 'click');
var example = source.takeUntil(click);     

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// complete (点击body了

```

[JSBin](https://jsbin.com/jogesut/2/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/ckyjuuva/)

这里我们一开始先用 `interval` 建立一个 observable，这个 observable 每隔 1 秒会发送一个从 0 开始递增的数值，接着我们用 `takeUntil`，传入另一个 observable。

当 `takeUntil` 传入的 observable 发送值时，原本的 observable 就会直接进入完成(complete)的状态，并且发送完成讯息。也就是说上面这段代码的行为，会先每 1 秒印出一个数字(从 0 递增)直到我们点击 body 为止，他才会发送 complete 讯息。

如果画成 Marble Diagram 则会像下面这样

```
source : -----0-----1-----2------3--
click  : ----------------------c----
                takeUntil(click)
example: -----0-----1-----2----|

```

当 click 一发送元素的时候，observable 就会直接完成(complete)。

### concatAll

有时我们的 Observable 发送的元素又是一个 observable，就像是二维阵列，阵列里面的元素是阵列，这时我们就可以用 `concatAll` 把它摊平成一维阵列，大家也可以直接把 concatAll 想成把所有元素 concat 起来。

```
var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.of(1,2,3));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/jogesut/6/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/ckyjuuva/4/)

这个示例我们每点击一次 body 就会立刻发送 1,2,3，如果用 Marble diagram 表示则如下

```
click  : ------c------------c--------

        map(e => Rx.Observable.of(1,2,3))

source : ------o------------o--------
                \            \
                 (123)|       (123)|

                   concatAll()

example: ------(123)--------(123)------------

```

这里可以看到 `source` observable 内部每次发送的值也是 observable，这时我们用 concatAll 就可以把 source 摊平成 example。

这里需要注意的是 `concatAll` 会处理 source 先发出来的 observable，必须等到这个 observable 结束，才会再处理下一个 source 发出来的 observable，让我们用下面这个示例说明。

```
var obs1 = Rx.Observable.interval(1000).take(5);
var obs2 = Rx.Observable.interval(500).take(2);
var obs3 = Rx.Observable.interval(2000).take(1);

var source = Rx.Observable.of(obs1, obs2, obs3);

var example = source.concatAll();

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
// 0
// 1
// 0
// complete

```

[JSBin](https://jsbin.com/jogesut/4/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/ckyjuuva/2/)

这里可以看到 `source` 会发送 3 个 observable，但是 `concatAll` 后的行为永远都是先处理第一个 observable，**等到当前处理的结束后才会再处理下一个**。

用 Marble diagram 表示如下

```
source : (o1                 o2      o3)|
           \                  \       \
            --0--1--2--3--4|   -0-1|   ----0|

                concatAll()        

example: --0--1--2--3--4-0-1----0|

```

## 简易拖拽

当学完前面几个 operator 后，我们就很轻松地做出拖拽的功能，先让我们来看一下需求

1.  首先画面上有一个元件(#drag)
2.  当滑鼠在元件(#drag)上按下左键(mousedown)时，开始监听滑鼠移动(mousemove)的位置
3.  当滑鼠左键放掉(mouseup)时，结束监听滑鼠移动
4.  当滑鼠移动(mousemove)被监听时，跟着修改元件的样式属性

第一步我已经完成了，大家可以直接到以下两个连结做练习

*   [JSBin](https://jsbin.com/yopawop/1/edit?js,output)
*   [JSFiddle](https://jsfiddle.net/s6323859/dc0se480/)

第二步我们要先取得各个 DOM 物件，元件(#drag) 跟 body。

```
const dragDOM = document.getElementById('drag');
const body = document.body;

```

要取得 body 的原因是因为滑鼠移动(mousemove)跟滑鼠左键放掉(mouseup)都应该是在整个 body 监听。

第三步我们写出各个会用到的监听事件，并用 `fromEvent` 来取得各个 observable。

*   对 #drag 监听 mousedown
*   对 body 监听 mouseup
*   对 body 监听 mousemove

```
const mouseDown = Rx.Observable.fromEvent(dragDOM, 'mousedown');
const mouseUp = Rx.Observable.fromEvent(body, 'mouseup');
const mouseMove = Rx.Observable.fromEvent(body, 'mousemove');

```

> 
> 
> 记得还没 `subscribe` 之前都不会开始监听，一定会等到 subscribe 之后 observable 才会开始送值。
> 
> 

第四步开始写逻辑

**当 mouseDown 时，转成 mouseMove 的事件**

```
const source = mouseDown.map(event => mouseMove)

```

**mouseMove 要在 mouseUp 后结束**

加上 `takeUntil(mouseUp)`

```
const source = mouseDown
               .map(event => mouseMove.takeUntil(mouseUp))

```

这时 source 大概长像这样

```
source: -------e--------------e-----
                \              \
                  --m-m-m-m|     -m--m-m--m-m|

```

> 
> 
> m 代表 mousemove event
> 
> 

用 `concatAll()` 摊平 source 成一维。

```
const source = mouseDown
               .map(event => mouseMove.takeUntil(mouseUp))
               .concatAll();                 

```

用 map 把 mousemove event 转成 x,y 的位置，并且订阅。

```
source
.map(m => {
    return {
        x: m.clientX,
        y: m.clientY
    }
})
.subscribe(pos => {
  	dragDOM.style.left = pos.x + 'px';
    dragDOM.style.top = pos.y + 'px';
})              

```

到这里我们就已经完成了简易的拖拽功能了!完整的代码如下

```
const dragDOM = document.getElementById('drag');
const body = document.body;

const mouseDown = Rx.Observable.fromEvent(dragDOM, 'mousedown');
const mouseUp = Rx.Observable.fromEvent(body, 'mouseup');
const mouseMove = Rx.Observable.fromEvent(body, 'mousemove');

mouseDown
  .map(event => mouseMove.takeUntil(mouseUp))
  .concatAll()
  .map(event => ({ x: event.clientX, y: event.clientY }))
  .subscribe(pos => {
  	dragDOM.style.left = pos.x + 'px';
    dragDOM.style.top = pos.y + 'px';
  })

```

不知道读者有没有感受到，我们整个代码不到 15 行，而且只要能够看懂各个 operators，我们程式可读性是非常的高。

虽然这只是一个简单的拖拽实现，但已经展示出 RxJS 带来的威力，它让我们的代码更加的简洁，也更好的维护！

> 
> 
> [这里](https://jsfiddle.net/s6323859/1ahzh7a7/2/)有完整的成果可以参考。
> 
> 

## 今日小结

我们今天介绍了四个 operators 分别是 take, first, takeUntil, concatAll，并且完成了一个简易的拖拽功能，我们之后会把这个拖拽功能做得更完整，并且整合其他功能！

不知道读者今天有没有收获？如果有任何问题，欢迎在下方留言给我！
如果你喜欢这篇文章，请至标题旁帮我按个 星星＋like，谢谢。