# 30 天精通 RxJS (11)： 真实示例 - 完整拖拽应用

> 
> 
> 有次不小心进到了优酷，发现优酷有个不错的功能，能大大的提升用户体验，就让我们一起来实现这个效果吧！
> 
> 

一样建议大家可以直接看影片

在第 08 篇的时候，我们已经成功做出简易的拖拽效果，今天要来做一个完整的应用，而且是实务上有机会遇到但不好处理的需求，那就是优酷的影片效果！

> 
> 
> 如果还没有用过优酷的读者可以先前往[这里](http://v.youku.com/v_show/id_XMTQ5NTg2MDk3Ng==.html?&f=26824174&from=y1.2-3.4.2&spm=a2h0j.8191423.item_XMTQ5NTg2MDk3Ng==.A)试用。
> 
> 

当我们在优酷看影片时往下滚动画面，影片会变成一个小视窗在右下角，这个视窗还能够拖拽移动位置。这个功能可以让使用者一边看留言同时又能看影片，且不影响其他的资讯显示，真的是很不错的 feature。

![优酷影片拖拽功能](https://res.cloudinary.com/dohtkyi84/image/upload/v1482841073/30days/youku_drag.png)

就让我们一起来实现这个功能，同时补完拖拽所需要注意的细节吧！

## 需求分析

首先我们会有一个影片在最上方，原本是位置是静态(static)的，卷轴滚动到低于影片高度后，影片改为相对于视窗的绝对位置(fixed)，往回滚会再变回原本的状态。当影片为 fixed 时，滑鼠移至影片上方(hover)会有遮罩(masker)与鼠标变化(cursor)，可以拖拽移动(drag)，且移动范围不超过可视区间！

上面可以拆分成以下几个步骤

*   准备 static 样式与 fixed 样式
*   HTML 要有一个固定位置的锚点(anchor)
*   当滚动超过锚点，则影片变成 fixed
*   当往回滚动过锚点上方，则影片变回 static
*   影片 fixed 时，要能够拖拽
*   拖拽范围限制在当前可视区间

基本的 HTML 跟 CSS 笔者已经帮大家完成，大家可以直接到下面的链接接着实现：

*   [JSBin](https://jsbin.com/pevozex/1/edit?html,css,js,output)
*   [JSFiddle](https://jsfiddle.net/s6323859/ochbtpk5/1/)

先让我们看一下 HTML，首先在 HTML 里有一个 div(#anchor)，这个 div(#anchor) 就是待会要做锚点用的，它内部有一个 div(#video)，则是滚动后要改变成 fixed 的元件。

CSS 的部分我们只需要知道滚动到下方后，要把 div(#video) 加上 `video-fixed` 这个 class。

接着我们就开始实现滚动的效果切换 class 的效果吧！

### 第一步，取得会用到的 DOM

因为先做滚动切换 class，所以这里用到的 DOM 只有 #video, #anchor。

```javascript
const video = document.getElementById('video');
const anchor = document.getElementById('anchor');

```

### 第二步，建立会用到的 observable

这里做滚动效果，所以只需要监听滚动事件。

```javascript
const scroll = Rx.Observable.fromEvent(document, 'scroll');

```

### 第三步，撰写程式逻辑

这里我们要取得了 scroll 事件的 observable，当滚过 #anchor 最底部时，就改变 #video 的 class。

首先我们会需要滚动事件发生时，去判断是否**滚过 #anchor 最底部**，所以把原本的滚动事件变成是否滚过最底部的 true or false。

```javascript
scroll.map(e => anchor.getBoundingClientRect().bottom < 0)

```

这里我们用到了 `getBoundingClientRect` 这个浏览器原生的 API，他可以取得 DOM 事件的宽高以及上下左右离萤幕可视区间上(左)的距离，如下图

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1482844440/30days/getBoundingClientRect.png)

当我们可视范围区间滚过 #anchor 底部时， `anchor.getBoundingClientRect().bottom` 就会变成负值，此时我们就改变 #video 的 class。

```javascript
scroll
.map(e => anchor.getBoundingClientRect().bottom < 0)
.subscribe(bool => {
    if(bool) {
        video.classList.add('video-fixed');
    } else {
        video.classList.remove('video-fixed');
    }
})

```

到这里我们就已经完成滚动变更样式的效果了！

全部的 JS 代码，如下

```javascript
const video = document.getElementById('video');
const anchor = document.getElementById('anchor');

const scroll = Rx.Observable.fromEvent(document, 'scroll');

scroll
.map(e => anchor.getBoundingClientRect().bottom < 0)
.subscribe(bool => {
    if(bool) {
        video.classList.add('video-fixed');
    } else {
        video.classList.remove('video-fixed');
    }
})

```

> 
> 
> 当然这段还能在用 debounce/throttle 或 requestAnimationFrame 做优化，这个部分我们日后的文章会在提及。
> 
> 

接下来我们就可以接着做**拖拽的行为**了。

### 第一步，取得会用到的 DOM

这里我们会用到的 DOM 跟前面是一样的(#video)，所以不用多做什么。

### 第二步，建立会用到的 observable

这里跟上次一样，我们会用到 mousedown, mouseup, mousemove 三个事件。

```javascript
const mouseDown = Rx.Observable.fromEvent(video, 'mousedown')
const mouseUp = Rx.Observable.fromEvent(document, 'mouseup')
const mouseMove = Rx.Observable.fromEvent(document, 'mousemove')

```

### 第三步，撰写程式逻辑

跟上次是差不多的，首先我们会点击 #video 元件，点击(mousedown)后要变成移动事件(mousemove)，而移动事件会在滑鼠放开(mouseup)时结束(takeUntil)

```javascript
mouseDown
.map(e => mouseMove.takeUntil(mouseUp))
.concatAll()

```

因为把 mouseDown observable 发送出来的**事件**换成了 mouseMove observable，所以变成了 observable(mouseDown) 送出 observable(mouseMove)。因此最后用 concatAll 把后面送出的元素变成 mouse move 的事件。

> 
> 
> 这段如果不清楚的可以回去看一下 08 篇的讲解
> 
> 

但这里会有一个问题，就是我们的这段拖拽事件其实只能做用到 video-fixed 的时候，所以我们要加上 `filter`

```javascript
mouseDown
.filter(e => video.classList.contains('video-fixed'))
.map(e => mouseMove.takeUntil(mouseUp))
.concatAll()

```

这里我们用 filter 如果当下 #video 没有 `video-dragable` class 的话，事件就不会送出。

再来我们就能跟上次一样，把 mousemove 事件变成 { x, y } 的事件，并订阅来改变 #video 元件

```javascript
mouseDown
    .filter(e => video.classList.contains('video-fixed'))
    .map(e => mouseMove.takeUntil(mouseUp))
    .concatAll()
    .map(m => {
        return {
            x: m.clientX,
            y: m.clientY
        }
    })
    .subscribe(pos => {
        video.style.top = pos.y + 'px';
        video.style.left = pos.x + 'px';
    })

```

到这里我们基本上已经完成了所有功能，其步骤跟 08 篇的方法是一样的，如果不熟悉的人可以回头看一下！

但这里有两个大问题我们还没有解决

1.  第一次拉动的时候会闪一下，不像优酷那么顺
2.  拖拽会跑出当前可视区间，跑上出去后就抓不回来了

让我们一个一个解决，首先第一个问题是因为我们的拖拽直接给元件滑鼠的位置(clientX, clientY)，而非给滑鼠相对移动的距离！

所以要解决这个问题很简单，我们只要把点击目标的左上角当作 (0,0)，并以此改变元件的样式，就不会有闪动的问题。

这个要怎么做呢？ 很简单，我们在昨天讲了一个 operator 叫做 withLatestFrom，我们可以用它来把 mousedown 与 mousemove 两个 Event 的值同时传入 callback。

```javascript
mouseDown
    .filter(e => video.classList.contains('video-fixed'))
    .map(e => mouseMove.takeUntil(mouseUp))
    .concatAll()
    .withLatestFrom(mouseDown, (move, down) => {
        return {
            x: move.clientX - down.offsetX,
            y: move.clientY - down.offsetY
        }
    })
    .subscribe(pos => {
        video.style.top = pos.y + 'px';
        video.style.left = pos.x + 'px';
    })

```

当我们能够同时得到 mousemove 跟 mousedown 的事件，接着就只要把 滑鼠相对可视区间的距离(client) 减掉点按下去时 滑鼠相对元件边界的距离(offset) 就行了。这时拖拽就不会先闪动一下囉！

> 
> 
> 大家只要想一下，其实 client - offset 就是元件相对于可视区间的距离，也就是他一开始没动的位置！
> 
> 

![offset&client](https://res.cloudinary.com/dohtkyi84/image/upload/v1482854816/30days/offset.png)

接着让我们解决第二个问题，拖拽会超出可视范围。这个问题其实只要给最大最小值就行了，因为需求的关系，这里我们的元件是相对可视居间的绝对位置(fixed)，也就是说

*   top 最小是 0
*   left 最小是 0
*   top 最大是**可视高度**扣掉**元件本身高度**
*   left 最大是**可视宽度**扣掉**元件本身宽度**

这里我们先宣告一个 function 来处理这件事

```javascript
const validValue = (value, max, min) => {
    return Math.min(Math.max(value, min), max)
}

```

第一个参数给原本要给的位置值，后面给最大跟最小，如果今天大于最大值我们就取最大值，如果今天小于最小值则取最小值。

再来我们就可以直接把这个问题解掉了

```javascript
mouseDown
    .filter(e => video.classList.contains('video-fixed'))
    .map(e => mouseMove.takeUntil(mouseUp))
    .concatAll()
    .withLatestFrom(mouseDown, (move, down) => {
        return {
            x: validValue(move.clientX - down.offsetX, window.innerWidth - 320, 0),
            y: validValue(move.clientY - down.offsetY, window.innerHeight - 180, 0)
        }
    })
    .subscribe(pos => {
        video.style.top = pos.y + 'px';
        video.style.left = pos.x + 'px';
    })

```

这里我偷懒了一下，直接写死元件的宽高(320, 180)，实际上应该用 `getBoundingClientRect` 计算是比较好的。

现在我们就完成整个应用囉！

> 
> 
> [这里](https://jsfiddle.net/s6323859/ochbtpk5/3/)有最后完成的结果。
> 
> 

## 今日结语

我们简单地用了不到 35 行的代码，完成了一个还算复杂的功能。更重要的是我们还保持了整支程式的可读性，让我们之后维护更加的轻松。

今天的练习就到这边结束了，不知道读者有没有收获呢？ 如果有任何问题欢迎在下方留言给我！

如果你喜欢本篇文章请帮我按个 star 。