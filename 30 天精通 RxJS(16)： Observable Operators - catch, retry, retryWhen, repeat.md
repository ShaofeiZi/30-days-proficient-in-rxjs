# 30 天精通 RxJS(16): Observable Operators - catch, retry, retryWhen, repeat

我们已经快把所有基本的转换(Transformation)、过滤(Filter)和合并(Combination)的 operators 讲完了。今天要讲错误处理(Error Handling)的 operators，错误处理是非同步行为中的一大难题，尤其有多个交错的非同步行为时，更容易凸显错误处理的困难。

就让我们一起来看看在 RxJS 中能如何处理错误吧！

## Operators

### catch

catch 是很常见的非同步错误处理方法，在 RxJS 中也能够直接用 catch 来处理错误，在 RxJS 中的 catch 可以回传一个 observable 来送出新的值，让我们直接来看示例：

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch(error => Rx.Observable.of('h'));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    

```

[JSBin](https://jsbin.com/nafusoq/16/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/10/)

这个示例我们每隔 500 毫秒会送出一个字串(String)，并用字串的方法 `toUpperCase()` 来把字串的英文字母改成大写，过程中可能未知的原因送出了一个数值(Number) `2` 导致发生例外(数值没有 toUpperCase 的方法)，这时我们在后面接的 catch 就能抓到错误。

catch 可以回传一个新的 Observable、Promise、Array 或任何 Iterable 的物件，来传送之后的元素。

以我们的例子来说最后就会在送出 `X` 就结束，画成 Marble Diagram 如下

```
source : ----a----b----c----d----2|
        map(x => x.toUpperCase())
         ----a----b----c----d----X|
        catch(error => Rx.Observable.of('h'))
example: ----a----b----c----d----h|         

```

这里可以看到，当错误发生后就会进到 catch 并重新处理一个新的 observable，我们可以利用这个新的 observable 来送出我们想送的值。

也可以在遇到错误后，让 observable 结束，如下

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch(error => Rx.Observable.empty());

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    

```

[JSBin](https://jsbin.com/nafusoq/15/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/9/)

回传一个 empty 的 observable 来直接结束(complete)。

另外 catch 的 callback 能接收第二个参数，这个参数会接收当前的 observalbe，我们可以回传当前的 observable 来做到重新执行，示例如下

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch((error, obs) => obs);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    

```

[JSBin](https://jsbin.com/nafusoq/14/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/8/)

这里可以看到我们直接回传了当前的 obserable(其实就是 example)来重新执行，画成 Marble Diagram 如下

```
source : ----a----b----c----d----2|
        map(x => x.toUpperCase())
         ----a----b----c----d----X|
        catch((error, obs) => obs)
example: ----a----b----c----d--------a----b----c----d--..

```

因为是我们只是简单的示范，所以这里会一直无限循环，实务上通常会用在断线重连的情境。

另上面的处理方式有一个简化的写法，叫做 `retry()`。

### retry

如果我们想要一个 observable 发生错误时，重新尝试就可以用 retry 这个方法，跟我们前一个讲示例的行为是一致

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retry();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

[JSBin](https://jsbin.com/nafusoq/13/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/7/)

通常这种无限的 `retry` 会放在即时同步的重新连接，让我们在连线断掉后，不断的尝试。另外我们也可以设定只尝试几次，如下

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retry(1);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 
// a
// b
// c
// d
// a
// b
// c
// d
// Error: TypeError: x.toUpperCase is not a function

```

[JSBin](https://jsbin.com/nafusoq/12/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/6/)

这里我们对 retry 传入一个数值 `1`，能够让我们只重复尝试 1 次后送出错误，画成 Marble Diagram 如下

```
source : ----a----b----c----d----2|
        map(x => x.toUpperCase())
         ----a----b----c----d----X|
                retry(1)
example: ----a----b----c----d--------a----b----c----d----X|

```

这种处理方式很适合用在 HTTP request 失败的场景中，我们可以设定重新发送几次后，再秀出错误讯息。

### retryWhen

RxJS 还提供了另一种方法 `retryWhen`，他可以把例外发生的元素放到一个 observable 中，让我们可以直接操作这个 observable，并等到这个 observable 操作完后再重新订阅一次原本的 observable。

这里我们直接来看代码

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retryWhen(errorObs => errorObs.delay(1000));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

[JSBin](https://jsbin.com/nafusoq/11/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/5/)

这里 retryWhen 我们传入一个 callback，这个 callback 有一个参数会传入一个 observable，这个 observable 不是原本的 observable(example) 而是例外事件送出的错误所组成的一个 observable，我们可以对这个由错误所组成的 observable 做操作，等到这次的处理完成后就会重新订阅我们原本的 observable。

这个示例我们是把错误的 observable 送出错误延迟 1 秒，这会使后面重新订阅的动作延迟 1 秒才执行，画成 Marble Diagram 如下

```
source : ----a----b----c----d----2|
        map(x => x.toUpperCase())
         ----a----b----c----d----X|
        retryWhen(errorObs => errorObs.delay(1000))
example: ----a----b----c----d-------------------a----b----c----d----...

```

从上图可以看到后续重新订阅的行为就被延后了，但实务上我们不太会用 retryWhen 来做重新订阅的延迟，通常是直接用 catch 做到这件事。这里只是为了示范 retryWhen 的行为，实务上我们通常会把 retryWhen 拿来做错误通知或是例外收集，如下

```javascript
var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retryWhen(
                errorObs => errorObs.map(err => fetch('...')));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

这里的 `errorObs.map(err => fetch('...'))` 可以把 errorObs 里的每个错误变成 API 的发送，通常这里个 API 会像是送讯息到公司的通讯频道(Slack 等等)，这样可以让工程师马上知道可能哪个 API 挂了，这样我们就能即时地处理。

> 
> 
> retryWhen 实际上是在背地里建立一个 Subject 并把错误放入，会在对这个 Subject 进行内部的订阅，因为我们还没有讲到 Subject 的观念，大家可以先把它当作 Observable 就好了，另外记得这个 observalbe 预设是无限的，如果我们把它结束，原本的 observable 也会跟着结束。
> 
> 

### repeat

我们有时候可能会想要 retry 一直重复订阅的效果，但没有错误发生，这时就可以用 repeat 来做到这件事，示例如下

```javascript
var source = Rx.Observable.from(['a','b','c'])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source.repeat(1);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// a
// b
// c
// a
// b
// c
// complete

```

[JSBin](https://jsbin.com/nafusoq/10/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/3/)

这里 repeat 的行为跟 retry 基本一致，只是 retry 只有在例外发生时才触发，画成 Marble Diagram 如下

```
source : ----a----b----c|
            repeat(1)
example: ----a----b----c----a----b----c|

```

同样的我们可以不给参数让他无限循环，如下

```
var source = Rx.Observable.from(['a','b','c'])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source.repeat();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/nafusoq/9/edit?js,console) | [JSFiddle](https://jsfiddle.net/aruku1xr/1/)

这样我们就可以做动不断重复的行为，这个可以在建立轮询时使用，让我们不断地发 request 来更新画面。

最后我们来看一个错误处理在实际应用中的小示例

```javascript
const title = document.getElementById('title');

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x)
            .map(x => x.toUpperCase()); 
            // 通常 source 会是建立即时同步的连线，像是 web socket

var example = source.catch(
                (error, obs) => Rx.Observable.empty()
                               .startWith('连线发生错误： 5秒后重连')
                               .concat(obs.delay(5000))
                 );

example.subscribe({
    next: (value) => { title.innerText = value },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

[JSBin](https://jsbin.com/nafusoq/6/edit?js,output) | [JSFiddle](https://jsfiddle.net/aruku1xr/)

这个示例其实就是模仿在即时同步断线时，利用 catch 返回一个新的 observable，这个 observable 会先送出错误讯息并且把原本的 observable 延迟 5 秒再做合并，虽然这只是一个模仿，但它清楚的展示了 RxJS 在做错误处理时的灵活性。

### 今日小结

今天我们讲了三个错误处理的方法还有一个 repeat operator，这几个方法都很有机会在实际上用到，不知道今天大家有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，谢谢！