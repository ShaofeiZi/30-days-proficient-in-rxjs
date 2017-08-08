# 30 天精通 RxJS(27)：简易实现 Observable(二)

> 
> 
> 原本这篇文章是要跟前一篇写一起的，但发现两篇加起来快 3000 字了所以拆成两篇。
> 
> 

前一篇文章我们已经完成了基本的 observable 以及 Observer 的简易实现，这篇文章我们会接续上一篇的内容实现简易的 Observable 类别，以及一个 creation operator 和一个 transform operator。

## 建立简易 Observable 类别

这是我们上一篇文章写的建立 observable 实例的函数

```javascript
function create(subscribe) {
    const observable = {
        subscribe: function(observerOrNext, error, complete) {
            const realObserver = new Observer(observerOrNext, error, complete)
            subscribe(realObserver);
            return realObserver;
        }       
    };
    return observable;
}

```

[JSBin](https://jsbin.com/paxevam/2/edit?js,console)

从这个函数可以看出来，回传的 observable 实例至少会有 subscribe 方法，所以最简单的 Observable 类别大概会长像下面这样

```javascript
class Observable {
  subscribe(observerOrNext, error, complete) {
    // ...做某些事
  }
}

```

另外 create 的函数在执行时会传入一个 subscribe 的 function，这个 function 会决定 observable 的行为

```javascript
var observable = create(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next('not work');
})

```

把上面这一段改成下面这样

```javascript
var observable = new Observable(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next('not work');
})

```

所以我们的 Observable 的建构式应该会接收一个 subscribe function

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    // ...做某些事
  }
}

```

接着我们就能完成 subscribe 要做的事情了

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到 _subscribe 属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    this._subscribe(observer); // 就是执行一个 function 对吧
    return observer;
  }
}

```

到这里我们就成功的把 create 的函数改成 Observable 的类别了，我们可以直接来使用看看

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    this._subscribe(observer);
    return observer;
  }
}

var observable = new Observable(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next('not work');
})

var observer = {
  next: function(value) {
    console.log(value)
  },
  complete: function() {
      console.log('complete!')
  }
}

observable.subscribe(observer);

```

[JSBin](https://jsbin.com/paxevam/3/edit?js,console)

当然我们可以仿 RxJS 在静态方法中加入 create，如下

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    this._subscribe(observer);
    return observer;
  }
}

Observable.create = function(subscribe) {
    return new Observable(subscribe);
}

```

这样一来我们就可以用 `Observable.create` 建立 observable 实例实例。

```javascript
var observable = Observable.create(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next('not work');
});

```

[JSBin](https://jsbin.com/paxevam/4/edit?js,console)

## 建立 creation operator - fromArray

当我们有 Observable 类别后要建立 creation operator 就不难了，这里我们建立一个 fromArray 的方法，可以接收 array 来建立 observable，算是 Rx 的 Observable.from 的简化版本，记得 creation operators 都属于 static 方法

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    this._subscribe(observer);
    return observer;
  }
}

// 建立静态方法 
Observable.fromArray = function(array) {
    if(!Array.isArray(array)) {
        // 如果传入的参数不是阵列，则抛出例外
        throw new Error('params need to be an array');
    }
    return new Observable(function(observer) {
        try{
            // 遍历每个元素并送出
            array.forEach(value => observer.next(value))
            observer.complete()
        } catch(err) {
            observer.error(err)
        }
    });
}

var observable = Observable.fromArray([1,2,3,4,5]);

```

[JSBin](https://jsbin.com/paxevam/6/edit?js,console)

上面的代码我们只是简单的用 new Observable 就可以轻松地实现我们要的功能，之后就可以用 fromArray 来建立 observable 实例。

相信读者到这之前应该都不会有太大的问题，接下来这个部份就困难的多，请读者们一定要搞懂前面的各个实现再接着往下看。

## 建立 transform operator - map

相信很多人在实现 Observable 都是卡在这个阶段，因为 operators 都是回传一个新的 observable 这中间有很多细节需要注意，并且有些小技巧才能比较好的实现，在开始实现之前先让我们厘清几个重点

*   operators(transform, filter, conditional...) 都是回传一个新个 observable
*   大部分的 operator 其实就是在原本 observer 外包裹一层实例，让执行 next 方法前先把元素做一次处理
*   operator 回传的 observable 订阅时，还是需要执行原本的 observable(资料源)，也就说我们要想办法保留原本的 observable

让我们一步一步来，首先 operators 执行完会回传一个新的 observable，这个 observable 在订阅时会先去执行 operator 的行为再发送元素，所以 observable 的订阅方法就不能像现在这样直接把 observer 传给 subscribe 执行

```javascript
class Observable {
  constructor(subscribe) {
    if(subscribe) {
      this._subscribe = subscribe; // 把 subscribe 存到属性中
    }
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    // 先做某个判断是否当前的 observable 是具有 operator 的
    if(??) {
      // 用 operator 的操作
    } else {
      // 如果没有 operator 再直接把 observer 丢给 _subscribe
      this._subscribe(observer);
    }
    return observer;
  }
}

```

> 
> 
> 以我们的 Observable 实现为例，这里最重要的就是 this._subscribe 执行，每当执行时就是开始发送元素。
> 
> 

这里我们可以想像一下当一个 map 产生的 observable 订阅时，应该先判断出有 map 这个 operator 并且传入原本的资料源以及当前的 observer。也就是说我们的 map 至少有以下这几件事要做

*   建立新的 observable
*   保存原本的 observable(资料源)，之后订阅时才有办法执行
*   建立并保存 operator 本身的行为，等到订阅时执行

```javascript
class Observable {
  constructor(subscribe) {
    // 一些代码...
  }
  subscribe(observerOrNext, error, complete) {
    // 一些代码...
  }
  map(callback) {
    const observable = new Observable(); // 建立新的 observable

    observable.source = this; // 保存当前的 observable(资料源)

    observable.operator = {
        call: (observer, source) => { // 执行这个 operator 的行为 }
    }; // 储存当前 operator 行为，并作为是否有 operator 的依据，

    return observable; // 返回这个新的 observable
  }
}

```

上面这三个步骤都是必要的，特别是用到了 `observable.source = this` 这个小技巧，来保存原本的 observable。但这里我们还有一个地方没完成就是 operator 要做的事，这个部分我们等一下再补，先把 subscribe 写完

```javascript
class Observable {
  constructor(subscribe) {
    // 一些代码...
  }
  subscribe(observerOrNext, error, complete) {
    const observer = new Observer(observerOrNext, error, complete);
    // 先用 this.operator 判断当前的 observable 是否具有 operator 
    if(this.operator) {
      this.operator.call(observer, this.source)
    } else {
      // 如果没有 operator 再直接把 observer 丢给 _subscribe
      this._subscribe(observer);
    }
    return observer;
  }
  map(callback) {
    const observable = new Observable(); // 建立新的 observable

    observable.source = this; // 保存当前的 observable(资料源)

    observable.operator = {
        call: (observer, source) => { // 执行这个 operator 的行为 }
    }; // 储存当前 operator 行为，并作为是否有 operator 的依据，

    return observable; // 返回这个新的 observable
  }
}

```

记得这里补的 subscribe 行为，已经是 map 回传新 observable 的行为，不是原本的 observable 了。

到这里我们就几乎要完成了，接着只要实现 map 这个 operator 的行为就可以囉！记得我们在前面讲的 operator 其实就是在原本的 observer 做一层包裹，让 next 执行前先对元素做处理，所以我们改写一下 Observer 并建立一个 MapObserver 来做这件事

```javascript
class Observer {
  constructor(destinationOrNext, error, complete) {
    switch (arguments.length) {
      case 0:
        this.destination = this.safeObserver(emptyObserver);
        break;
      case 1:
        if (!destinationOrNext) {
          this.destination = this.safeObserver(emptyObserver);
          break;
        }
        // 多一个判断，是否传入的 destinationOrNext 原本就是 Observer 的实例，如果是就不用在用执行 `this.safeObserver`
        if(destinationOrNext instanceof Observer){
          this.destination = destinationOrNext;
          break;
        }
        if (typeof destinationOrNext === 'object') {
          this.destination = this.safeObserver(destinationOrNext);
          break;
        }
      default:
        this.destination = this.safeObserver(destinationOrNext, error, complete);
        break;
    }
  }

  // ...下面都一样
}

class MapObserver extends Observer {
  constructor(observer, callback) {
    // 这里会传入原本的 observer 跟 map 的 callback
    super(observer); // 因为有继承所以要先执行一次父层的建构式
    this.callback = callback; // 保存 callback
    this.next = this.next.bind(this); // 确保 next 的 this
  }
  next(value) {
    try {
      this.destination.next(this.callback(value)); 
      // this.destination 是父层 Observer 保存的 observer 实例
      // 这里 this.callback(value) 就是 map 的操作
    } catch (err) {
      this.destination.error(err);
      return;
    }
  }
}

```

上面这段代码就可以让我们包裹 observer 实例，利用实例的继承覆写原本的 next 方法。

最后我们就只要补完 map 方法就可以了

```javascript
class Observable {
  constructor(subscribe) {
    // 一些代码...
  }
  subscribe(observerOrNext, error, complete) {
    // 一些代码...
  }
  map(callback) {
    const observable = new Observable(); 
    observable.source = this;
    observable.operator = {
      call: (observer, source) => { 
        // 执行这个 operator 的行为
        const newObserver = new MapObserver(observer, callback);
        // 建立包裹后的 observer
        // 订阅原本的资料源，并回传
        return source.subscribe(newObserver);
      }
    };    
    return observable; 
  }
}

```

这里做的事情就简单很多，我们只要建立包裹过的 observer，并用这个包裹后的 observer 订阅原本的 source。(记得这个 function 是在 subscribe 时执行的)

[这里有](https://jsbin.com/paxevam/7/edit?js,console)完整的代码，可以让大家参考。

另外这里有抽出 lift 方法的实现，其实跟我们现在的版本很接近了，只是把建立新的 observable 封装到 lift 而已。

## 今日小结

今天这篇文章介绍了要如何简易的实现 Observable，虽然说是简易版本但实际上已经非常非常接近 RxJS 官方的实现了，希望读者花点耐心一步一步跟着代码做，做出来后再慢慢吸收。

不知道今天读者们有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，谢谢！