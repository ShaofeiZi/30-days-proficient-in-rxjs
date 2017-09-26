# 30 天精通 RxJS(18): Observable Operators - switchMap, mergeMap, concatMap

今天我们要讲三个非常重要的 operators，这三个 operators 在很多的 RxJS 相关的 library 的使用示例上都会看到。很多初学者在使用这些 library 时，看到这三个 operators 很可能就放弃了，但其实如果有把这个系列的文章完整看过的话，现在应该就能很好接受跟理解。

## Operators

### concatMap

concatMap 其实就是 map 加上 concatAll 的简化写法，我们直接来看一个示例

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .concatAll();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

上面这个示例就可以简化成

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .concatMap(
                    e => Rx.Observable.interval(100).take(3)
                );

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

前后两个行为是一致的，记得 concatMap 也会先处理前一个送出的 observable 在处理下一个 observable，画成 Marble Diagram 如下

```
source : -----------c--c------------------...
        concatMap(c => Rx.Observable.interval(100).take(3))
example: -------------0-1-2-0-1-2---------...

```

这样的行为也很常被用在发送 request 如下

```
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nuhita/5/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/t2zxtuh0/2/)

这里我们每点击一下画面就会送出一个 HTTP request，如果我们快速的连续点击，大家可以在开发者工具的 network 看到每个 request 是等到前一个 request 完成才会送出下一个 request，如下图

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483454601/30days/concatMap_request.png)

> 
> 
> 这里建议把网速模拟调到最慢
> 
> 

![](http://res.cloudinary.com/dohtkyi84/image/upload/v1483503231/30days/throttle_network.png)

从 network 的图形可以看得出来，第二个 request 的发送时间是接在第一个 request 之后的，我们可以确保每一个 request 会等前一个 request 完成才做处理。

concatMap 还有第二个参数是一个 selector callback，这个 callback 会传入四个参数，分别是

1.  外部 observable 送出的元素
2.  内部 observable 送出的元素
3.  外部 observable 送出元素的 index
4.  内部 observable 送出元素的 index

回传值我们想要的值，示例如下

```
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                e => Rx.Observable.from(getPostData()), 
                (e, res, eIndex, resIndex) => res.title);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nuhita/7/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/t2zxtuh0/3/)

这个示例的外部 observable 送出的元素就是 click event 实例，内部 observable 送出的元素就是 response 实例，这里我们回传 response 实例的 title 属性，这样一来我们就可以直接收到 title，这个方法很适合用在 response 要选取的值跟前一个事件或顺位(index)相关时。

### switchMap

switchMap 其实就是 map 加上 switch 简化的写法，如下

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .switch();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

上面的代码可以简化成

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .switchMap(
                    e => Rx.Observable.interval(100).take(3)
                );

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

画成 Marble Diagram 表示如下

```
source : -----------c--c-----------------...
        concatMap(c => Rx.Observable.interval(100).take(3))
example: -------------0--0-1-2-----------...

```

只要注意一个重点 switchMap 会在下一个 observable 被送出后直接退订前一个未处理完的 observable，这个部份的细节请看上一篇文章 switch 的部分。

另外我们也可以把 switchMap 用在发送 HTTP request

```
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.switchMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nuhita/1/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/t2zxtuh0/4/)

如果我们快速的连续点击五下，可以在开发者工具的 network 看到每个 request 会在点击时发送，如下图

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483456745/30days/switchMap_request.png)

> 
> 
> 灰色是浏览器原生地停顿行为，实际上灰色的一开始就是 fetch 执行送出 request，只是卡在浏览器等待发送。
> 
> 

从上图可以看到，虽然我们发送了多个 request 但最后真正印出来的 log 只会有一个，代表前面发送的 request 已经不会造成任何的 side-effect 了，这个很适合用在只看最后一次 request 的情境，比如说 自动完成(auto complete)，我们只需要显示使用者最后一次打在画面上的文字，来做建议选项而不用每一次的。

switchMap 跟 concatMap 一样有第二个参数 selector callback 可用来回传我们要的值，这部分的行为跟 concatMap 是一样的，这里就不再赘述。

### mergeMap

mergeMap 其实就是 map 加上 mergeAll 简化的写法，如下

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .mergeAll();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

上面的代码可以简化成

```
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .mergeMap(
                    e => Rx.Observable.interval(100).take(3)
                );

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

画成 Marble Diagram 表示

```
source : -----------c-c------------------...
        concatMap(c => Rx.Observable.interval(100).take(3))
example: -------------0-(10)-(21)-2----------...

```

记得 mergeMap 可以并行处理多个 observable，以这个例子来说当我们快速点按两下，元素发送的时间点是有机会重叠的，这个部份的细节大家可以看上一篇文章 merge 的部分。

另外我们也可以把 switchMap 用在发送 HTTP request

```
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.mergeMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nuhita/3/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/t2zxtuh0/5/)

如果我们快速的连续点击五下，大家可以在开发者工具的 network 看到每个 request 会在点击时发送并且会 log 出五个实例，如下图

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483457934/30days/mergeMap_request.png)

mergeMap 也能传入第二个参数 selector callback，这个 selector callback 跟 concatMap 第二个参数也是完全一样的，但 mergeMap 的重点是我们可以传入第三个参数，来限制并行处理的数量

```
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.mergeMap(
                e => Rx.Observable.from(getPostData()), 
                (e, res, eIndex, resIndex) => res.title, 3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nuhita/4/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/t2zxtuh0/5/)

这里我们传入 3 就能限制，HTTP request 最多只能同时送出 3 个，并且要等其中一个完成在处理下一个，如下图

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483458530/30days/mergeMap_3_request.png)

大家可以注意看上面这张图，我连续点按了五下，但第四个 request 是在第一个完成后才送出的，这个很适合用在特殊的需求下，可以限制同时发送的 request 数量。

> 
> 
> RxJS 5 还保留了 mergeMap 的别名叫 flatMap，虽然官方文件上没有，但这两个方法是完全一样的。请参考[这里](https://github.com/ReactiveX/RxJS/issues/333)
> 
> 

### switchMap, mergeMap, concatMap

这三个 operators 还有一个共同的特性，那就是这三个 operators 可以把第一个参数所回传的 promise 实例直接转成 observable，这样我们就不用再用 `Rx.Observable.from` 转一次，如下

```
function getPersonData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(e => getPersonData());
                                    //直接回传 promise 实例

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

至於在使用上要如何选择这三个 operators？ 其实都还是看使用情境而定，这里笔者简单列一下大部分的使用情境

*   concatMap 用在可以确定**内部的 observable 结束时间比外部 observable 发送时间来快的情境**，并且不希望有任何并行处理行为，适合少数要一次一次完成到底的的 UI 动画或特别的 HTTP request 行为。
*   switchMap 用在只要最后一次行为的结果，适合绝大多数的使用情境。
*   mergeMap 用在并行处理多个 observable，适合需要并行处理的行为，像是多个 I/O 的并行处理。

> 
> 
> 建议初学者不确定选哪一个时，使用 switchMap
> 
> 

> 
> 
> 在使用 concatAll 或 concatMap 时，请注意内部的 observable 一定要能够的结束，且外部的 observable 发送元素的速度不能比内部的 observable 结束时间快太多，不然会有 memory issues
> 
> 

## 今日小结

今天的文章内容主要讲了三个 operators，如果有看完上一篇文章的读者应该不难吸收，主要还是使用情境上需要思考以及注意一些细节。

不知道今天读者有没有收获呢？ 如果有任何问题，欢迎留言给我，谢谢