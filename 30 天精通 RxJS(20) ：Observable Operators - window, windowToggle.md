# 30 天精通 RxJS(20): Observable Operators - window, windowToggle, groupBy

前几天我们讲完了能把 Higher Order Observable 转成一般的 Observable 的 operators，今天我们要讲能够把一般的 Observable 转成 Higher Order Observable 的 operators。

其实前端不太有机会用到这类型的 Operators，都是在比较特殊的需求下才会看到，但还是会有遇到的时候。

## Operators

### window

window 是一整个家族，总共有五个相关的 operators

*   window
*   windowCount
*   windowTime
*   windowToggle
*   windowWhen

这里我们只介绍 window 跟 windowToggle 这两个方法，其他三个的用法相对都简单很多，大家如果有需要可以再自行到官网查看。

window 很类似 buffer 可以把一段时间内送出的元素拆出来，只是 buffer 是把元素拆分到数组中变成

```javascript
Observable<T> => Observable<Array<T>>

```

而 window 则是会把元素拆分出来放到新的 observable 变成

```javascript
Observable<T> => Observable<Observable<T>>

```

buffer 是把拆分出来的元素放到数组并送出数组；window 是把拆分出来的元素放到 observable 并送出 observable，让我们来看一个例子

```javascript
var click = Rx.Observable.fromEvent(document, 'click');
var source = Rx.Observable.interval(1000);
var example = source.window(click);

example
  .switch()
  .subscribe(console.log);
// 0
// 1
// 2
// 3
// 4
// 5 ...

```

首先 window 要传入一个 observable，每当这个 observable 送出元素时，就会把正在处理的 observable 所送出的元素放到新的 observable 中并送出，这里看 Marble Diagram 会比较好解释

```javascript
click  : -----------c----------c------------c--
source : ----0----1----2----3----4----5----6---..
                    window(click)
example: o----------o----------o------------o--
         \          \          \
          ---0----1-|--2----3--|-4----5----6|
                    switch()
       : ----0----1----2----3----4----5----6---... 

```

这里可以看到 example 变成发送 observable 会在每次 click 事件发送出来后结束，并继续下一个 observable，这里我们用 switch 才把它摊平。

当然这个范例只是想单存的表达 window 的作用，没什么太大的意义，实务上 window 会搭配其他的 operators 使用，例如我们想计算一秒钟内触发了几次 click 事件

```javascript
var click = Rx.Observable.fromEvent(document, 'click');
var source = Rx.Observable.interval(1000);
var example = click.window(source)

example
  .map(innerObservable => innerObservable.count())
  .switch()
  .subscribe(console.log);

```

[JSBin](https://jsbin.com/fudocigewi/4/edit?html,js,output) | [JSFiddle](https://jsfiddle.net/sy1fybre/3/)

注意这里我们把 source 跟 click 对调了，并用到了 observable 的一个方法 `count()`，可以用来取得 observable 总共送出了几个元素，用 Marble Diagram 表示如下

```
source : ---------0---------1---------2--...
click  : --cc---cc----c-c----------------...
                    window(source)
example: o--------o---------o---------o--..
         \        \         \         \
          -cc---cc|---c-c---|---------|--..
                    count()
       : o--------o---------o---------o--
         \        \         \         \
          -------4|--------2|--------0|--..
                    switch()
       : ---------4---------2---------0--... 

```

从 Marble Diagram 中可以看出来，我们把部分元素放到新的 observable 中，就可以利用 Observable 的方法做更灵活的操作

## windowToggle

windowToggle 不像 window 只能控制内部 observable 的结束，windowToggle 可以传入两个参数，第一个是开始的 observable，第二个是一个 callback 可以回传一个结束的 observable，让我们来看范例

```javascript
var source = Rx.Observable.interval(1000);
var mouseDown = Rx.Observable.fromEvent(document, 'mousedown');
var mouseUp = Rx.Observable.fromEvent(document, 'mouseup');

var example = source
  .windowToggle(mouseDown, () => mouseUp)
  .switch();

example.subscribe(console.log);

```

[JSBin](https://jsbin.com/fudocigewi/3/edit?html,js,output) | [JSFiddle](https://jsfiddle.net/sy1fybre/2/)

一样用 Marble Diagram 会比较好解释

```
source   : ----0----1----2----3----4----5--...

mouseDown: -------D------------------------...
mouseUp  : ---------------------------U----...

        windowToggle(mouseDown, () => mouseUp)

         : -------o-------------------------...
                  \
                   -1----2----3----4--|
                   switch()
example  : ---------1----2----3----4---------...                                     

```

从 Marble Diagram 可以看得出来，我们用 windowToggle 拆分出来内部的 observable 始于 mouseDown 终于 mouseUp。

### groupBy

最后我们来讲一个开发上比较常用的 operators - groupBy，它可以帮我们把相同条件的元素拆分成一个 Observable，其实就跟平常在 SQL 下是一样个概念，我们先来看个简单的例子

```javascript
var source = Rx.Observable.interval(300).take(5);

var example = source
              .groupBy(x => x % 2);

example.subscribe(console.log);

// GroupObservable { key: 0, ...}
// GroupObservable { key: 1, ...}

```

[JSBin](https://jsbin.com/fudocigewi/1/edit?html,js,console) | [JSFiddle](https://jsfiddle.net/sy1fybre/1/)

上面的例子，我们传入了一个 callback function 并回传 groupBy 的条件，就能区分每个元素到不同的 Observable 中，用 Marble Diagram 表示如下

```
source : ---0---1---2---3---4|
             groupBy(x => x % 2)
example: ---o---o------------|
            \   \
            \   1-------3----|
            0-------2-------4|

```

在实际上，我们可以拿 groupBy 做完元素的区分后，再对 inner Observable 操作，例如下面这个例子我们将每个人的分数作加总再送出

```javascript
var people = [
    {name: 'Anna', score: 100, subject: 'English'},
    {name: 'Anna', score: 90, subject: 'Math'},
    {name: 'Anna', score: 96, subject: 'Chinese' }, 
    {name: 'Jerry', score: 80, subject: 'English'},
    {name: 'Jerry', score: 100, subject: 'Math'},
    {name: 'Jerry', score: 90, subject: 'Chinese' }, 
];
var source = Rx.Observable.from(people)
						   .zip(
						     Rx.Observable.interval(300), 
						     (x, y) => x);

var example = source
  .groupBy(person => person.name)
  .map(group => group.reduce((acc, curr) => ({ 
	    name: curr.name,
	    score: curr.score + acc.score 
	})))
	.mergeAll();

example.subscribe(console.log);
// { name: "Anna", score: 286 }
// { name: 'Jerry', score: 270 }

```

[JSBin](https://jsbin.com/fudocigewi/2/edit?html,js,console) | [JSFiddle](https://jsfiddle.net/sy1fybre/)

这里我们范例是想把 Jerry 跟 Anna 的分数个别作加总，画成 Marble Diagram 如下

```
source : --o--o--o--o--o--o|

  groupBy(person => person.name)

       : --i--------i------|
           \        \
           \         o--o--o|
            o--o--o--|

	   map(group => group.reduce(...))

       : --i---------i------|
           \         \
           o|        o|

             mergeAll()
example: --o---------o------|           

```

## 今日小结

今天讲了两个可以把元素拆分到新的 observable 的 operators，这两个 operators 在前端比较少用到，但在后端或是比较复杂了前端应用才比较有机会用到。不知道读者有没有收获呢？ 如果有任何问题欢迎留言给我，谢谢。