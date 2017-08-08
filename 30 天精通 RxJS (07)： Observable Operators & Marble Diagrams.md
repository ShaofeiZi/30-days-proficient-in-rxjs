# 30 天精通 RxJS (07)： Observable Operators & Marble Diagrams

> 
> 
> Observable 的 Operators 是实例应用上最重要的部份，我们需要了解各种 Operators 的使用方式，才能轻松实现各种需求！
> 
> 

这是【30天精通 RxJS】的 07 篇，如果还没看过 06 篇可以往这边走：
[30 天精通 RxJS (06)： 建立 Observable(二)](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(06)%EF%BC%9A%20%E5%BB%BA%E7%AB%8B%20Observable(%E4%BA%8C).md)

昨天我们把所有建立 Observable 实例的 operators 讲完了，接下来我们要讲关于转换(Transformation)、过滤(Filter)、合并(Combination)等操作方法。先来让我们看看什么是 Operator

## 什么是 Operator？

Operators 就是一个个被附加到 Observable 型别的函数，例如像是 map, filter, contactAll... 等等，所有这些函数都会拿到原本的 observable 并回传一个新的 observable，就像有点像下面这个样子

```
var people = Rx.Observable.of('Jerry', 'Anna');

function map(source, callback) {
    return Rx.Observable.create((observer) => {
        return source.subscribe(
            (value) => { 
                try{
                    observer.next(callback(value));
                } catch(e) {
                    observer.error(e);
                }
            },
            (err) => { observer.error(err); },
            () => { observer.complete() }
        )
    })
}

var helloPeople = map(people, (item) => item + ' Hello~');

helloPeople.subscribe(console.log);
// Jerry Hello~
// Anna Hello~

```

[JSBin](https://jsbin.com/roginet/1/edit?js,console,output) | [JSFiddle](https://jsfiddle.net/s6323859/Lruuusf0/)

这里可以看到我们写了一个 map 的函数，它接收了两个参数，第一个是原本的 observable，第二个是 map 的 callback function。map 内部第一件事就是用 `create` 建立一个新的 observable 并回传，并且在内部订阅原本的 observable。

当然我们也可以直接把 map 塞到 `Observable.prototype`

```
function map(callback) {
    return Rx.Observable.create((observer) => {
        return this.subscribe(
            (value) => { 
                try{
                    observer.next(callback(value));
                } catch(e) {
                    observer.error(e);
                }
            },
            (err) => { observer.error(err); },
            () => { observer.complete() }
        )
    })
}
Rx.Observable.prototype.map = map;
var people = Rx.Observable.of('Jerry', 'Anna');
var helloPeople = people.map((item) => item + ' Hello~');

helloPeople.subscribe(console.log);
// Jerry Hello~
// Anna Hello~

```

这里有两个重点是我们一定要知道的，每个 operator 都会回传一个新的 observable，而我们可以透过 `create` 的方法建立各种 operator。

> 
> 
> 在 RxJS 5 的实例中，其实每个 operator 是透过原来 observable 的 lift 方法来建立新的 observable，这个方法会在新回传的 observable 事件内偷塞两个属性，分别是 source（来源） 与 operator，记录原本的资料源跟当前使用的 operator。
> 
> 

> 
> 
> 其实 lift 方法还是用 new Observable(跟 create 一样)。至于为什么要独立出这个方法，除了更好的封装以外，主要的原因是为了让 RxJS 5 的使用者能更好的 debug。关于 RxJS 5 的除错方式，我们会专门写一篇来讲解！
> 
> 

> 
> 
> 这里我们只是简单的实例 operator。如果之后实例上，想要不影响原本的 Observable 又能够自订 operator 可以参考官方的这份[文件](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md)。(现在先不用看)
> 
> 

其实 RxJS 提供的各种 operators 已经非常够用了，不太需要我们自己创造 operator，这里只是想让大家先对 operator 的建立有个基本的观念，之后在学习的过程中会比较轻松。

在我们开始介绍 RxJS 的 operators 前，为了能让我们更好地理解各种 operators，我们需要先订定一个简单的方式来表达 observable！

## Marble diagrams

我们在传达事物时，文字其实是最糟的手段，虽然文字是我们平时沟通的基础，但常常千言万语也比不过一张清楚的图。如果我们能订定 observable 的图示，就能让我们更方便的沟通及理解 observable 的各种 operators！

我们把描绘 observable 的图示称为 Marble diagrams，在网路上 RxJS 有非常多的 Marble diagrams，规则大致上都是相同的，这里为了方便撰写以及跟读者的留言互动，所以采用类似 ASCII 的绘画方式。

我们用 `-` 来表达一小段时间，这些 `-` 串起就代表一个 observable。

```
----------------

```

`X` (大写 X)则代表有错误发生

```
---------------X

```

`|` 则代表 observable 结束

```
----------------|

```

在这个时间序当中，我们可能会发发送值(value)，如果值是数字则直接用阿拉伯数字取代，其他的资料型别则用相近的英文符号代表，这里我们用 `interval` 举例

```
var source = Rx.Observable.interval(1000);

```

`source` 的图形就会长像这样

```
-----0-----1-----2-----3--...

```

当 observable 是同步送值的时候，例如

```
var source = Rx.Observable.of(1,2,3,4);

```

`source` 的图形就会长像这样

```
(1234)|

```

小括号代表着同步发生。

另外的 Marble diagrams 也能够表达 operator 的前后转换，例如

```
var source = Rx.Observable.interval(1000);
var newest = source.map(x => x + 1); 

```

这时 Marble diagrams 就会长像这样

```
source: -----0-----1-----2-----3--...
            map(x => x + 1)
newest: -----1-----2-----3-----4--...

```

最上面是原本的 observable，中间是 operator，下面则是新的 observable。

以上就是 Marble diagrams 如何表示 operator 对 observable 的操作，这能让我们更好的理解各个 operator。

> 
> 
> Marble Diagrams 相关资源：<http://rxmarbles.com/>
> 
> 

最后让我们来看几个简单的 Operators！

## Operators

### map

Observable 的 map 方法使用上跟数组的 map 是一样的，我们传入一个 callback function，这个 callback function 会带入每次发发送来的元素，然后我们回传新的元素，如下

```
var source = Rx.Observable.interval(1000);
var newest = source.map(x => x + 1); 

newest.subscribe(console.log);
// 1
// 2
// 3
// 4
// 5..

```

用 Marble diagrams 表达就是

```
source: -----0-----1-----2-----3--...
            map(x => x + 1)
newest: -----1-----2-----3-----4--...

```

我们有另外一个方法跟 map 很像，叫 mapTo

### mapTo

mapTo 可以把传进来的值改成一个固定的值，如下

```
var source = Rx.Observable.interval(1000);
var newest = source.mapTo(2); 

newest.subscribe(console.log);
// 2
// 2
// 2
// 2..

```

mapTo 用 Marble diagrams 表达

```
source: -----0-----1-----2-----3--...
                mapTo(2)
newest: -----2-----2-----2-----2--...

```

### filter

filter 在使用上也跟数组的相同，我们要传入一个 callback function，这个 function 会传入每个被发送的元素，并且回传一个 boolean 值，如果为 true 的话就会保留，如果为 false 就会被滤掉，如下

```
var source = Rx.Observable.interval(1000);
var newest = source.filter(x => x % 2 === 0); 

newest.subscribe(console.log);
// 0
// 2
// 4
// 6..

```

filter 用 Marble diagrams 表达

```
source: -----0-----1-----2-----3-----4-...
            filter(x => x % 2 === 0)
newest: -----0-----------2-----------4-...

```

> 
> 
> 读者应该有发现 map, filter 这些方法其实都跟数组的相同，因为这些都是 functional programming 的通用函数，就算换个语言也有机会看到相同的命名及相同的用法。
> 
> 

> 
> 
> 实际上 Observable 跟 Array 的 operators(map, filter)，在行为上还是有极大的差异。当我们的资料量很大时，Observable 的效能会好上非常多。我们会有一天专门讲这个部份！
> 
> 

## 今日小结

今天我们讲了 Observable Operators 的相关知识，有以下几个重点

*   什么是 Operators
    *   如何建立 operator
*   Marble diagrams
*   Operators
    *   map
    *   mapTo
    *   filter

不知道今天读者有没有收获呢？欢迎在下方留言给我，这是精通 RxJS 的第 07 篇！