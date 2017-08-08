# 30 天精通 RxJS(25)：Subject 总结

Subject 其实在 RxJS 中最常被误解的一部份，因为 Subject 可以让你用命令式的方式虽送值到一个 observable 的串流中。很多人会直接把这个特性拿来用在 **不知道如何建立 Observable 的状况**，比如我们在 [30 天精通 RxJS(23)](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS(23)%EF%BC%9ASubject%2C%20BehaviorSubject%2C%20ReplaySubject%2C%20AsyncSubject.md)中提到的可以用在 ReactJS 的 Event 中，来建立 event 的 observable

```javascript
class MyButton extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
        this.subject = new Rx.Subject();

        this.subject
            .mapTo(1)
            .scan((origin, next) => origin + next)
            .subscribe(x => {
                this.setState({ count: x })
            })
    }
    render() {
        return <button onClick={event => this.subject.next(event)}>{this.state.count}</button>
    }
}

```

因为在 React API 的关系，如果我们想要把 React Event 转乘 observable 就可以用 Subject 帮我们做到这件事；但绝大多数的情况我们是可以透过 `Observable.create` 来做到这件事，像下面这样

```javascript
const example = Rx.Observable.creator(observer => {
    const source = getSomeSource(); // 某个资料源
    source.addListener('some', (some) => {
        observer.next(some)
    })
});

```

大概就会像上面这样，如果没有合适的 creation operators 我们还是可以利用 `Observable.create` 来建立 observable，除非真的因为框架限制才会直接用 Subject。

## Subject 与 Observable 的差异

永远记得 Subject 其实是 Observer Design Pattern 的实例，所以当 observer 订阅到 subject 时，subject 会把订阅者塞到一份订阅者清单，在元素发送时就是在遍历这份清单，并把元素一一送出，这跟 Observable 像是一个 function 执行是完全不同的(请参考 05 篇)。

Subject 之所以具有 Observable 的所有方法，是因为 Subject 继承了 Observable 的型别，其实 Subject 型别中**主要**实做的方法只有 next、error、 complete、subscribe 及 unsubscribe 这五个方法，而这五个方法就是依照 Observer Pattern 下去实现的。

总而言之，Subject 是 Observable 的子类别，这个子类别当中用上述的五个方法实例了 Observer Pattern，所以他同时具有 Observable 与 Observer 的特性，而跟 Observable 最大的差异就是 Subject 是具有状态的，也就是储存的那份清单！

## 当前版本会遇到的问题

因为 Subject 在订阅时，是把 observer 放到一份清单当中，并在元素要送出(next)的时候遍历这份清单，大概就像下面这样

```javascript
//...
next() {
    // observers 是一个阵列存有所有的 observer 
    for (let i = 0; i < observers.length; i++) {
        observers[i].next(value);
    }
}
//...

```

这会衍伸一个大问题，就是在某个 observer 发生错误却没有做错误处理时，就会影响到别的订阅，看下面这个例子

```javascript
const source = Rx.Observable.interval(1000);
const subject = new Rx.Subject();

const example = subject.map(x => {
    if (x === 1) {
        throw new Error('oops');
    }
    return x;
});
subject.subscribe(x => console.log('A', x));
example.subscribe(x => console.log('B', x));
subject.subscribe(x => console.log('C', x));

source.subscribe(subject);

```

[JSBin](https://jsbin.com/hukalo/1/edit?html,js,console)

上面这个例子，大家可能会预期 B 会在送出 1 的时候挂掉，另外 A 跟 C 则会持续发送元素，确实正常应该像这样运席；但目前 RxJS 的版本中会在 B 报错之后，A 跟 C 也同时停止运行。原因就像我前面所提的，在遍历所有 observer 时发生了例外会导致之后的行为停止。

> 
> 
> 这个应该会在之后的版本中改掉的，前阵子才在 [TC39 Observable proposal](https://github.com/tc39/proposal-observable/issues/119#issuecomment-269429238) 中讨论完。
> 
> 

那要如何解决这个问题呢？ 目前最简单的方式当然是尽可能地把所有 observer 的错误处理加进去，这样一来就不会有例外发生

```javascript
const source = Rx.Observable.interval(1000);
const subject = new Rx.Subject();

const example = subject.map(x => {
    if (x === 1) {
        throw new Error('oops');
    }
    return x;
});
subject.subscribe(
    x => console.log('A', x),
    error => console.log('A Error:' + error));
example.subscribe(x => console.log('B', x),
    error => console.log('B Error:' + error));
subject.subscribe(x => console.log('C', x),
    error => console.log('C Error:' + error));

source.subscribe(subject);

```

[JSBin](https://jsbin.com/hukalo/2/edit?html,js,console)

像上面这段代码，当 B 发生错误时就只有 B 会停止，而不会影响到 A 跟 C。

当然还有另一种解法是用 Scheduler，但因为我们这系列的文章还没有讲到 Scheduler 所以这个解法大家看看就好

```javascript
const source = Rx.Observable.interval(1000);
const subject = new Rx.Subject().observeOn(Rx.Scheduler.asap);

const example = subject.map(x => {
    if (x === 1) {
        throw new Error('oops');
    }
    return x;
});
subject.subscribe(x => console.log('A', x));
example.subscribe(x => console.log('B', x));
subject.subscribe(x => console.log('C', x));

source.subscribe(subject);

```

## 一定需要使用 Subject 的时机？

Subject 必要的使用时机除了本篇文章一开始所提的之外，正常应该是当我们一个 observable 的操作过程中发生了 side-effect 而我们不希望这个 side-effect 因为多个 subscribe 而被触发多次，比如说下面这段代码

```javascript
var result = Rx.Observable.interval(1000).take(6)
             .map(x => Math.random()); // side-effect，平常有可能是呼叫 API 或其他 side effect

var subA = result.subscribe(x => console.log('A: ' + x));
var subB = result.subscribe(x => console.log('B: ' + x));

```

[JSBin](https://jsbin.com/bogiful/2/edit?html,js,console)

这段代码 A 跟 B 印出来的乱数就不一样，代表 random(side-effect) 被执行了两次，这种情况就一定会用到 subject(或其相关的 operators)

```javascript
var result = Rx.Observable.interval(1000).take(6)
             .map(x => Math.random()) // side-effect
             .multicast(new Rx.Subject())
             .refCount();

var subA = result.subscribe(x => console.log('A: ' + x));
var subB = result.subscribe(x => console.log('B: ' + x));

```

[JSBin](https://jsbin.com/bogiful/1/edit?html,js,console)

改成这样后我们就可以让 side-effect 不会因为订阅数而多执行，这种情状就是一定要用 subject 的。

## 今日小结

今天总结了 Subject 的使用情境，以及厘清跟 Observable 的关系，并且指出在使用时要避免犯发生的错误。

这几点都非常的重要，不知道今天读者有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，谢谢！