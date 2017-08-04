# 30 天精通 RxJS (13)： Observable Operator - delay, delayWhen

> 
> 
> 在所有非同步中行为中，最麻烦的大概就是 UI 操作了，因为 UI 是直接影响使用者的感受，如果处理的不好对使用体验会大大的扣分！
> 
> 

UI 大概是所有非同步行为中最不好处理的，不只是因为它直接影响了用户体验，更大的问题是 UI 互动常常是高频率触发的事件，而且多个元件间的时间序需要不一致，要做到这样的 UI 互动就不太可能用 Promise 或 async/await，但是用 RxJS 仍然能轻易地处理！

今天我们要介绍的两个 Operators，delay 跟 delayWhen 都是跟 UI 互动比较相关的。当我们的网页越来越像应用程式时，UI 互动就变得越重要，让我们来试试如何用 RxJS 完成基本的 UI 互动！

## Operators

### delay

delay 可以延迟 observable 一开始发送元素的时间点，示例如下

```javascript
var source = Rx.Observable.interval(300).take(5);

var example = source.delay(500);

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

```

[JSBin](https://jsbin.com/qodegan/1/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/pnjw51o5/)

当然直接从 log 出的讯息看，是完全看不出差异的

让我们直接看 Marble Diagram

```
source : --0--1--2--3--4|
        delay(500)
example: -------0--1--2--3--4|

```

从 Marble Diagram 可以看得出来，第一次送出元素的时间变慢了，虽然在这里看起来没什么用，但是在 UI 操作上是非常有用的，这个部分我们最后示范。

delay 除了可以传入毫秒以外，也可以传入 Date 型别的资料，如下使用方式

```javascript
var source = Rx.Observable.interval(300).take(5);

var example = source.delay(new Date(new Date().getTime() + 1000));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/qodegan/2/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/pnjw51o5/1/)

~~这好像也能用在预定某个日期，让程式挂掉233~~

## delayWhen

delayWhen 的作用跟 delay 很像，最大的差别是 delayWhen 可以影响每个元素，而且需要传一个 callback 并回传一个 observable，示例如下

```javascript
var source = Rx.Observable.interval(300).take(5);

var example = source
              .delayWhen(
                  x => Rx.Observable.empty().delay(100 * x * x)
              );

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[JSBin](https://jsbin.com/qodegan/3/edit?js,console) | [JSFiddle](https://jsfiddle.net/s6323859/pnjw51o5/2/)

这时我们的 Marble Diagram 如下

```
source : --0--1--2--3--4|
    .delayWhen(x => Rx.Observable.empty().delay(100 * x * x));
example: --0---1----2-----3-----4|

```

这里传进来的 x 就是 source 送出的每个元素，这样我们就能对每一个做延迟。

这里我们用 delay 来做一个小功能，这个功能很简单就是让多张照片跟着滑鼠跑，但每张照片不能跑一样快！

首先我们准备六张大头照，并且写进 HTML

```html
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover6.jpg" alt="">
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover5.jpg" alt="">
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover4.jpg" alt="">
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover3.jpg" alt="">
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover2.jpg" alt="">
<img src="https://res.cloudinary.com/dohtkyi84/image/upload/c_scale,w_50/v1483019072/head-cover1.jpg" alt="">

```

用 CSS 把 img 改成圆形，并加上边筐以及绝对位置

```css
img{
  position: absolute;
  border-radius: 50%;
  border: 3px white solid;
  transform: translate3d(0,0,0);
}

```

再来写 JS，一样第一步先抓 DOM

```javascript
var imgList = document.getElementsByTagName('img');

```

第二步建立 observable

```javascript
var movePos = Rx.Observable.fromEvent(document, 'mousemove')
.map(e => ({ x: e.clientX, y: e.clientY }))

```

第三步撰写逻辑

```javascript
function followMouse(DOMArr) {
  const delayTime = 600;
  DOMArr.forEach((item, index) => {
    movePos
      .delay(delayTime * (Math.pow(0.65, index) + Math.cos(index / 4)) / 2)
      .subscribe(function (pos){
        item.style.transform = 'translate3d(' + pos.x + 'px, ' + pos.y + 'px, 0)';
      });
  });
}

followMouse(Array.from(imgList))

```

这里我们把 imgList 从 Collection 转成 Array 后传入 `followMouse()`，并用 forEach 把每个 omg 取出并利用 index 来达到不同的 delay 时间，这个 delay 时间的逻辑大家可以自己想，不用跟我一样，最后 subscribe 就完成啦！

最后完整的示例在[这里](https://jsbin.com/hayixa/2/edit?html,css,js,output)

## 今日小结

今天我们介绍了两个 operators 并带了一个小示例，这两个 operators 在 UI 操作上都非常的实用，我们明天会接着讲几个 operators 可以用来做高频率触发的事件优化！