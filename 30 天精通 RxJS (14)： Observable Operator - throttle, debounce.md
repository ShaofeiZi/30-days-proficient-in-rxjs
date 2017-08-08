# 30 天精通 RxJS (14)： Observable Operator - throttle, debounce

昨天讲到了在 UI 操作上很常用的 delay，今天我们接着要来讲另外两个也非常实用 operators，尤其在做性能优化时更是不可或缺的好工具！

## Operators

### debounce

跟 buffer、bufferTime 一样，Rx 有 debounce 跟 debounceTime 一个是传入 observable 另一个则是传入毫秒，比较常用到的是 debounceTime，这里我们直接来看一个示例

```javascript
var source = Rx.Observable.interval(300).take(5);
var example = source.debounceTime(1000);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 4
// complete

```

[JSBin](https://jsbin.com/nemepo/5/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/kqwk0yvp/1/)

这里只印出 `4` 然后就结束了，因为 **debounce 运作的方式是每次收到元素，他会先把元素 cache 住并等待一段时间，如果这段时间内已经没有收到任何元素，则把元素送出；如果这段时间内又收到新的元素，则会把原本 cache 住的元素释放掉并重新计时，不断反复。**

以现在这个示例来讲，我们每 300 毫秒就会送出一个数值，但我们的 debounceTime 是 1000 毫秒，也就是说每次 debounce 收到元素还等不到 1000 毫秒，就会收到下一个新元素，然后重新等待 1000 毫秒，如此重复直到第五个元素送出时，observable 结束(complete)了，debounce 就直接送出元素。

以 Marble Diagram 表示如下

```
source : --0--1--2--3--4|
        debounceTime(1000)
example: --------------4|        

```

debounce 会在收到元素后等待一段时间，这很适合用来处理**间歇行为**，间歇行为就是指这个行为是一段一段的，例如要做 Auto Complete 时，我们要打字搜寻不会一直不断的打字，可以等我们停了一小段时间后再送出，才不会每打一个字就送一次 request！

这里举一个简单的例子，假设我们想要自动传送使用者打的字到后端

```javascript
const searchInput = document.getElementById('searchInput');
const theRequestValue = document.getElementById('theRequestValue');

Rx.Observable.fromEvent(searchInput, 'input')
  .map(e => e.target.value)
  .subscribe((value) => {
    theRequestValue.textContent = value;
    // 在这里发 request
  })

```

如果用上面这段代码，就会每打一个字就送一次 request，当很多人在使用时就会对 server 造成很大的负担，实际上我们只需要使用者最后打出来的文字就好了，不用每次都送，这时就能用 debounceTime 做优化。

```javascript
const searchInput = document.getElementById('searchInput');
const theRequestValue = document.getElementById('theRequestValue');

Rx.Observable.fromEvent(searchInput, 'input')
  .debounceTime(300)
  .map(e => e.target.value)
  .subscribe((value) => {
    theRequestValue.textContent = value;
    // 在这里发 request
  })

```

[JSBin](https://jsbin.com/nemepo/2/edit?js,output) | [JSFiddle](https://jsfiddle.net/s6323859/kqwk0yvp/2/)

这里建议大家到 JSBin 亲手试试，可以把 `debounceTime(300)` 注解掉，看看前后的差异。

### throttle

基本上每次看到 debounce 就会看到 throttle，他们两个的作用都是要降低事件的触发频率，但行为上有很大的不同。

跟 debounce 一样 RxJS 有 throttle 跟 throttleTime 两个方法，一个是传入 observable 另一个是传入毫秒，比较常用到的也是 throttleTime，让我们直接来看示例

```javascript
var source = Rx.Observable.interval(300).take(5);
var example = source.throttleTime(1000);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 4
// complete

```

[JSBin](https://jsbin.com/nemepo/6/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/kqwk0yvp/)

跟 debounce 的不同是 throttle 会先开放送出元素，等到有元素被送出就会沉默一段时间，等到时间过了又会开放发送元素。

throttle 比较像是控制行为的最高频率，也就是说如果我们设定 **1000 毫秒**，那该事件频率的最大值就是**每秒触发一次**不会再更快，debounce 则比较像是必须等待的时间，要等到一定的时间过了才会收到元素。

throttle 更适合用在**连续性行为**，比如说 UI 动画的运算过程，因为 UI 动画是连续的，像我们之前在做拖拉时，就可以加上 `throttleTime(12)` 让 mousemove event 不要发送的太快，避免画面更新的速度跟不上样式的切换速度。

> 
> 
> 浏览器有一个 [requestAnimationFrame](https://developer.mozilla.org/zh-TW/docs/Web/API/Window.requestAnimationFrame) API 是专门用来优化 UI 运算的，通常用这个的效果会比 throttle 好，但并不是绝对还是要看最终效果。
> 
> 

> 
> 
> RxJS 也能用 requestAnimationFrame 做优化，而且使用方法很简单，这个部份会在 Scheduler 提到。
> 
> 

## 今日小结

今天介绍了两个非常实用的方法，可以帮我们做程式的性能优化，而且使用方式非常的简单，不知道读者有没有收获？ 如果有任何问题，欢迎在下方留言给我，谢谢。