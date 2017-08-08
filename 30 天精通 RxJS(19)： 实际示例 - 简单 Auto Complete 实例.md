# 30 天精通 RxJS(19)： 实际示例 - 简单 Auto Complete 实例



今天我们要做一个 RxJS 的经典示例 - 自动完成 (Auto Complete)，自动完成在开发上的应用非常广泛，几乎随处可见这样的功能，只要是跟表单、搜寻相关的都会看到。

虽然是个很常见的功能，但多数的工程师都只是直接套插件来完成，很少有人会自己从头到尾把完整的逻辑写一次。如果有自己写过这个功能的工程师，应该就会知道这个功能在实现的过程中很多细节会让代码变的非常复杂，像是要如何取消上一次发送出去的 request、要如何优化请求次数... 等等，这些小细节都会让代码变的非常复杂且很难维护。

就让我们一起来用 RxJS 来实现这个功能吧！

## 需求分析

首先我们会有一个搜索框(input#search)，当我们在上面打字并停顿超过 100 毫秒就发送 HTTP Request 来取得建议选项并显示在收寻框下方(ul#suggest-list)，如果使用者在前一次发送的请求还没有回来就打了下一个字，此时前一个发送的请求就要舍弃掉，当建议选项显示之后可以用滑鼠点击取建议选项代搜索框的文字。

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483543558/30days/autocomplete.png)

上面的叙述可以拆分成以下几个步骤

*   准备 input#search 以及 ul#suggest-list 的 HTML 与 CSS
*   在 input#search 输入文字时，等待 100 毫秒再无输入，就发送 HTTP Request
*   当 Response 还没回来时，使用者又输入了下一个文字就舍弃前一次的并再发送一次新的 Request
*   接受到 Response 之后显示建议选项
*   滑鼠点击后取代 input#search 的文字

基本的 HTML 跟 CSS 笔者已经帮大家完成，大家可以直接到下面的连结接着实现：

*   [JSBin](https://jsbin.com/yaxupi/3/edit?js,output)

先让我们看一下 HTML，首先在 HTML 里有一个 input(#search)，这个 input(#search) 就是要用来输入的栏位，它下方有一个 ul(#suggest-list)，则是放建议选项的地方

CSS 的部分可以不用看，JS 的部分已经写好了要发送 API 的 url 跟方法`getSuggestList`，接着就开始实现自动完成的效果吧！

### 第一步，取得需要的 DOM 事件

这里我们会用到 #search 以及 #suggest-list 这两个 DOM

```javascript
const searchInput = document.getElementById('search');
const suggestList = document.getElementById('suggest-list');

```

### 第二步，建立所需的 Observable

这里我们要监听 搜索栏位的 input 事件，以及建议选项的点击事件

```javascript
const keyword = Rx.Observable.fromEvent(searchInput, 'input');
const selectItem = Rx.Observable.fromEvent(suggestList, 'click');

```

### 第三步，撰写代码逻辑

每当使用者输入文字就要发送 HTTP request，并且有新的值被输入后就舍弃前一次发送的，所以这里用 switchMap

```javascript
keyword.switchMap(e => getSuggestList(e.target.value))

```

这里我们先试着订阅，看一下 API 会回传什么样的资料

```javascript
keyword
    .switchMap(e => getSuggestList(e.target.value))
    .subscribe(console.log)

```

在 search 栏位乱打几个字

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483545742/30days/wikires.png)

大家可以在 console 看到资料长相这样，他会回传一个数组带有四个元素，其中第一个元素是我们输入的值，第二个元素才是我们要的建议选项清单。

所以我们要取的是 response 数组的第二的元素，用 switchMap 的第二个参数来选取我们要的

```javascript
keyword
    .switchMap(
        e => getSuggestList(e.target.value),
        (e, res) => res[1]
    )
    .subscribe(console.log)

```

这时再输入文字就可以看到确实是我们要的返回值

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483546009/30days/wikirealres.png)

写一个 render 方法，把数组转成 li 并写入 suggestList

```javascript
const render = (suggestArr = []) => {
    suggestList.innerHTML = suggestArr
                            .map(item => '<li>'+ item +'</li>')
                            .join('');  
}

```

这时我们就可用 render 方法把取得的数组传入

```javascript
const render = (suggestArr = []) => {
    suggestList.innerHTML = suggestArr
                            .map(item => '<li>'+ item +'</li>')
                            .join('');  
}

keyword
  .switchMap(
    e => getSuggestList(e.target.value),
    (e, res) => res[1]
  )
  .subscribe(list => render(list))

```

如此一来我们打字就能看到结果出现在 input 下方了

![](https://res.cloudinary.com/dohtkyi84/image/upload/v1483543558/30days/autocomplete.png)

只是目前还不能点选，先让我们来做点选的功能，这里点选的功能我们需要用到 delegation event 的小技巧，利用 ul 的 click 事件，来筛选是否点到了 li，如下

```javascript
selectItem
  .filter(e => e.target.matches('li'))

```

上面我们利用 DOM 事件的 matches 方法(里面的字串放 css 的 selector)来过滤出有点击到 li 的事件，再用 map 转出我们要的值并写入 input。

```javascript
selectItem
  .filter(e => e.target.matches('li'))
  .map(e => e.target.innerText)
  .subscribe(text => searchInput.value = text)

```

现在我们就能点击建议清单了，但是点击后清单没有消失，这里我们要在点击后重新 redner，所以把上面的代码改一下

```javascript
selectItem
  .filter(e => e.target.matches('li'))
  .map(e => e.target.innerText)
  .subscribe(text => { 
      searchInput.value = text;
      render();
  })

```

这样一来我们就完成最基本的功能了，大家可以到[这里](https://jsbin.com/yaxupi/6/edit?js,output)看初步的完成品。

还记得我们前面说每次打完字要等待 100 毫秒在发送 request 吗？ 这样能避免过多的 request 发送，可以降低 server 的负载也会有比较好的使用者体验，要做到这件事很简单只要加上 `debounceTime(100)` 就完成了

```javascript
keyword
  .debounceTime(100)
  .switchMap(
    e => getSuggestList(e.target.value),
    (e, res) => res[1]
  )
  .subscribe(list => render(list))

```

当然这个数值可以依照需求或是请 UX 针对这个细节作调整。

这样我们就完成所有功能了，大家可以到[这里](https://jsbin.com/yaxupi/7/edit?js,output)查看结果。

## 今日小结

我们用了不到 30 行的代码就完成了 auto complete 的基本功能，当我们能够自己从头到尾的完成这样的功能，在面对各种不同的需求，我们就能很方便的针对需求作调整，而不会受到插件的限制！比如说我们希望使用者打了 2 个字以上在发送 request，这时我们只要加上一行 filter 就可以了

```javascript
keyword
  .filter(e => e.target.value.length > 2)
  .debounceTime(100)
  .switchMap(
    e => getSuggestList(e.target.value),
    (e, res) => res[1]
  )
  .subscribe(list => render(list))

```

又或者网站的使用量很大，可能 API 在量大的时候会回传失败，主管希望可以在 API 失败的时候重新尝试 3 次，我们只要加个 `retry(3)` 就完成了

```javascript
keyword
  .filter(e => e.target.value.length > 2)
  .debounceTime(100)
  .switchMap(
    e => Rx.Observable.from(getSuggestList(e.target.value))
                      .retry(3),
    (e, res) => res[1]
  )
  .subscribe(list => render(list))

```

大家会发现我们的灵活度变的非常高，又同时兼顾了代码的可读性，短短的几行代码就完成了一个复杂的需求，这就是 RxJS 的魅力啊～