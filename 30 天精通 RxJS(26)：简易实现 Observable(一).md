# 30 天精通 RxJS(26)：简易实例 Observable(一)

因为实在太多读者在问要如何使用 Observable，所以特别调整了本系列文章最后几篇的内容，空出一天的位置来写如何简易实现 Observable。

为什么是简易实例而不完整实例呢？ 当然这个系列的文章是希望读者能学会如何使用 RxJS，而 实使用 Observable 其实只是帮助我们理解 Observable 的运作方式，所以这篇文章会尽可能地简单，一来让读者容易理解及吸收，二来有兴趣的读者可以再沿着这篇文章的内容去完整的实现。

## 重点观念

Observable 跟 Observer Pattern 是不同的，Observable 内部并没有管理一份订阅清单，**订阅 Observable 就像是执行一个 function 一样**！

所以实现过程的重点

*   订阅就是执行一个 funciton
*   订阅接收的实例具备 next, error, complete 三个方法
*   订阅会返回一个可退订(unsubscribe)的实例

## 基本 observable 实现

先用最简单的 function 来建立 observable 实例

```javascript
function create(subscriber) {
    var observable = {
        subscribe: function(observer) {
            subscriber(observer)
        }       
    };
    return observable;
}

```

上面这段代码就可以做最简单的订阅，像下面这样

```javascript
function create(subscriber) {
    var observable = {
        subscribe: function(observer) {
            subscriber(observer)
        }       
    };
    return observable;
}

var observable = create(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
})

var observer = {
  next: function(value) {
    console.log(value)
  }
}

observable.subscribe(observer)
// 1
// 2
// 3

```

[JSBin](https://jsbin.com/tububez/1/edit?js,console)

这时我们已经有最简单的功能了，但这里有一个大问题，就是 observable 在结束(complete)就不应该再发送元素

```javascript
var observable = create(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next('still work');
})

var observer = {
  next: function(value) {
    console.log(value)
  },
  complete: function() {
    console.log('complete!')
  }
}

observable.subscribe(observer)
// 1
// 2
// 3
// "complete!"
// "still work"

```

[JSBin](https://jsbin.com/tububez/2/edit?js,console)

从上面的代码可以看到 complete 之后还是能送元素出来，另外还有一个问题就是 observer，如果是不完整的就会出错，这也不是我们希望看到的。

```javascript
var observable = create(function(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete(); // error: complete is not a function 
})

var observer = {
  next: function(value) {
    console.log(value)
  }
}

observable.subscribe(observer)
// 1
// 2
// 3
// "complete!"
// "still work"

```

[JSBin](https://jsbin.com/tububez/3/edit?js,console)

上面这段代码可以看出来，当使用者 observer 实例没有 complete 方法时，就会报错。
我们应该修正这两个问题！

## 实现简易 Observer

要修正这两个问题其实并不难，我们只要实现一个 Observer 的类别，每次使用者传入的 observer 都会利用这个类别转乘我们想要 Observer 实例。

首先订阅时有可能传入一个 observer 实例，或是一到三个 function(next, error, complete)，所以我们要建立一个类别可以接受各种可能的参数

```javascript
class Observer {
  constructor(destinationOrNext, error, complete) {
    switch (arguments.length) {
      case 0:
        // 空的 observer
      case 1:
        if (!destinationOrNext) {
          // 空的 observer
        }
        if (typeof destinationOrNext === 'object') {
          // 传入了 observer 实例
        }
      default:
        // 如果上面都不是，代表应该是传入了一到三个 function
        break;
    }
  }
}

```

写一个方法(safeObserver)来回传正常的 observer

```javascript
class Observer {
  constructor(destinationOrNext, error, complete) {
    // ... 一些代码
  }
  safeObserver(observerOrNext, error, complete) {
    let next;

    if (typeof (observerOrNext) === 'function') {
      // observerOrNext 是 next function
      next = observerOrNext;
    } else if (observerOrNext) {
      // observerOrNext 是 observer 实例
      next = observerOrNext.next || () => {};
      error = observerOrNext.error || function(err) { 
        throw err 
      };
      complete = observerOrNext.complete || () => {};
    }
    // 最后回传我们预期的 observer 实例
    return {
      next: next,
      error: error,
      complete: complete
    };
  }
}

```

再把 constructor 完成

```javascript
// 预设空的 observer 
const emptyObserver = {
  next: () => {},
  error: (err) => { throw err; },
  complete: () => {}
}

class Observer {
  constructor(destinationOrNext, error, complete) {
    switch (arguments.length) {
      case 0:
        // 空的 observer
        this.destination = this.safeObserver(emptyObserver);
        break;
      case 1:
        if (!destinationOrNext) {
          // 空的 observer
          this.destination = this.safeObserver(emptyObserver);
          break;
        }
        if (typeof destinationOrNext === 'object') {
          // 传入了 observer 实例
          this.destination = this.safeObserver(destinationOrNext);
          break;
        }
      default:
        // 如果上面都不是，代表应该是传入了一到三个 function
        this.destination = this.safeObserver(destinationOrNext, error, complete);
        break;
    }
  }
  safeObserver(observerOrNext, error, complete) {
    // ... 一些代码
  }
}

```

[JSBin](https://jsbin.com/tububez/5/edit?js,console)

这里我们把真正的 observer 塞到 `this.destination`，接着完成 observer 的方法。

Observer 的三个主要的方法(next, error, complete)都应该结束或退订后不能再被执行，所以我们在实例内部偷塞一个 boolean 值来作为是否曾经结束的依据。

```javascript
class Observer {
  constructor(destinationOrNext, error, complete) {
    // ... 一些代码
  }
  safeObserver(observerOrNext, error, complete) {
    // ... 一些代码
  }
  unsubscribe() {
    this.isStopped = true; // 偷塞一个属性 isStopped
  }
}

```

接着要实现三个主要的方法就很简单了，只要先判断 `isStopped` 在使用 `this.destination` 实例来传送值就可以了

```javascript
class Observer {
  constructor(destinationOrNext, error, complete) {
    // ... 一些代码
  }
  safeObserver(observerOrNext, error, complete) {
    // ... 一些代码
  }

  next(value) {
    if (!this.isStopped && this.next) {
      // 先判断是否停止过
      try {
        this.destination.next(value); // 传送值
      } catch (err) {
        this.unsubscribe();
        throw err;
      }
    }
  }

  error(err) {
    if (!this.isStopped && this.error) {
      // 先判断是否停止过
      try {
        this.destination.error(err); // 传送错误
      } catch (anotherError) {
        this.unsubscribe();
        throw anotherError;
      }
      this.unsubscribe();
    }
  }

  complete() {
    if (!this.isStopped && this.complete) {
      // 先判断是否停止过
      try {
        this.destination.complete(); // 发送停止讯息
      } catch (err) {
        this.unsubscribe();
        throw err;
      }
      this.unsubscribe(); // 发送停止讯息后退订
    }
  }

  unsubscribe() {
    this.isStopped = true;
  }
}

```

[JSBin](https://jsbin.com/tububez/6/edit?js,console)

到这里我们就完成基本的 Observer 实现了，接着让我们拿到基本版的 observable 中使用吧。

```javascript
function create(subscriber) {
    const observable = {
        subscribe: function(observerOrNext, error, complete) {
            const realObserver = new Observer(observerOrNext, error, complete)
            subscriber(realObserver);
            return realObserver;
        }       
    };
    return observable;
}

var observable = create(function(observer) {
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
// 1
// 2
// 3
// complete!

```

[JSBin](https://jsbin.com/tububez/7/edit?js,console)

到这里我们就完成最基本的 observable 了，至少基本的行为都跟我们期望的一致，我知道读者们仍然不会放过我，你们会希望做出一个 Observable 型别以及至少一个 operator 对吧？ 不用担心，我们下一篇就会讲解如何建立一个 Observable 型别和 operator 的方法！

## 今日小结

今天我们复习了 Observable 的重要概念，并用这些重要的概念实现出了基本的 observable 以及 Observer 的类别。

不知道今天读者们有没有收获呢？ 如果有任何问题，欢迎在下方留言给我，谢谢！ 