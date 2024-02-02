---
title: '写一个符合 Promises/A+ 规范并可配合 ES7 async/await 使用的 Promise'
date: 2016/10/30
tag: javascript
author: Andy
---
[原文地址](https://www.ftandy.com/2016/10/30/mimi-promise/)

- - -

从历史的进程来看，Javascript 异步操作的基本单位已经从 callback 转换到 Promise。除了在特殊场景使用 [stream](https://github.com/request/request#streaming)，[RxJs](https://github.com/Reactive-Extensions/RxJS) 等更加抽象和高级的异步数据获取和操作流程方式外，现在几乎任何成熟的异步操作库，都会实现或者引用 Promise 作为 API 的返回单位。主流的 Javascript 引擎也基本原生实现了 Promise。

在 Promise 远未流行以前，Javascript 的异步操作基本都在使用以 callback 为主的异步接口。鼠标键盘事件，页面渲染，网络请求，文件请求等等异步操作的回调函数都是用 callback 来处理。随着异步使用场景范围的扩大，出现了大量工程化和富应用的的交互和操作，使得应用不足以用 callback 来面对愈加复杂的需求，慢慢出现了许多优雅和先进的异步解决方案：`EventEmitter`，`Promise`，`Web Worker`，`Generator`，`Async/Await`。

目前 Javascript 在客户端和服务器端的应用当中，只有 Promise 被广泛接受并使用。追根溯源，Promise 概念的提出是在 1976 年，Javascript 最早的 Promise 实现是在 2007 年，到现在 2016 年，Promise/A+ 规范和 ECMAscript 规范提出的 API 也足够稳定。`then`, `reject`, `all`, `spread`, `race`, `finally` 都是工程师开发中经常用到的 Promise API。很多人刚接触 Promise 概念的时候看下 API，看几篇博客或者看几篇最佳实践就以为理解程度够了，但是对 Promise 内部的异步机制不明了，使得在开发过程中遇到不少坑或者懵逼。

本文旨在让读者能深入了解 Promise 内部执行机制，熟悉和掌握 Promise 的操作流。如有兴趣，可以继续往下读。

<!--more-->

### Promise 只是一个 Event Loop 中的 microtask

深入了解过 Promise 的人都知道，Promise 所说的异步执行，只是将 Promise 构造函数中 `resolve`，`reject` 方法和注册的 callback 转化为 eventLoop的 microtask/Promise Job，并放到 Event Loop 队列中等待执行，也就是 Javascript 单线程中的“异步执行”。

[Promise/A+ 规范](https://promisesaplus.com/#point-67)中，并没有明确是以 microtask 还是 macrotask 形式放入队列，对没有 microtask 概念的宿主环境采用 setTimeout 等 task/Job 类的任务。规范中另外明确的一点也非常重要：回调函数的异步调用必须在当前 context，也就是 JS stack 为空之后执行。

在最新的 [ECMAScript 规范](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-promisereactionjob) 中，明确了 Promise 必须以 Promise Job 的形式入 Job 队列（也就是 microtask），并仅在没有运行的 stack(stack 为空的情况下)才可以初始化执行。

[HTML 规范](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint) 也提出，在 stack 清空后，执行 microtask 的检查方法。也就是必须在 stack 为空的情况下才能执行。

Google Chrome 的开发者 Jake Archibald （ES6-promise 作者）的文章 [Tasks, microtasks, queues and schedules](https://www.ftandy.com/2015/08/23/tasks-microtasks-queues-and-schedules/)中，将这个区分的问题描述得很清楚。假如要在 Javascript 平台或者引擎中实现 Promise，优先以 microtask/Promise Job 方式实现。目前主流浏览器的 Javascript 引擎原生实现，主流的 Promise 库（es6-promise，bluebrid）基本都是使用 microtask/Promise Job 的形式将 Promise 放入队列。

其他以 microtask/Promise Job 形式实现的方法还有：`process.nextTick`，`setImmediate`，`postMessag`e，`MessageChannel` 等

根据规范，microtask 存在的意义是：在当前 task 执行完，准备进行 I/O，`repaint`，`redraw` 等原生操作之前，需要执行一些低延迟的异步操作，使得浏览器渲染和原生运算变得更加流畅。这里的低延迟异步操作就是 microtask。原生的 setTimeout 就算是将延迟设置为 0 也会有 4 ms 的延迟，会将一个完整的 task 放进队列延迟执行，而且每个 task 之间会进行渲染等原生操作。假如每执行一个异步操作都要重新生成一个 task，将提高宿主平台的负担和响应时间。所以，需要有一个概念，在进行下一个 task 之前，将当前 task 生成的低延迟的，与下一个 task 无关的异步操作执行完，这就是 microtask。

这里的 [Quick Sort Demo](http://jphpsf.github.com/setImmediate-shim-demo) 展示了 microtask 和 task 在延迟执行上的巨大区别。

对于在不通宿主环境中选择合适的 microtask，可以选择 [asap](https://github.com/kriskowal/asap) 和 [setImmediate](https://github.com/YuzuJS/setImmediate) 的代码作为参考。
### Promise 的中的同步与异步

```javascript
new Promise((resolve) => {
  console.log('a')
  resolve('b')
  console.log('c')
}).then((data) => {
  console.log(data)
})

// a, c, b
```
使用过 Promise 的人都知道输出 a, c, b，但有多少人可以清楚地说出从创建 Promise 对象到执行完回调的过程？下面是一个完整的解释：

**构造函数中的输出执行是同步的，输出 a, 执行 `resolve` 函数，将 Promise 对象状态置为 `resolved`，输出 c。同时注册这个 Promise 对象的回调 `then` 函数。整个脚本执行完，stack 清空。event loop 检查到 stack 为空，再检查 microtask 队列中是否有任务，发现了 Promise 对象的 `then` 回调函数产生的 microtask，推入 stack，执行。输出 b，event loop的列队为空，stack 为空，脚本执行完毕。**

### 以基础的 Promises/A+ 规范为范本
规范地址：

- [Promises/A+](http://www.ituring.com.cn/article/66566) 规范（中文）
- [Promises/A+](https://promisesaplus.com/) 规范（英文）

值得注意的是：

{% blockquote %}

Finally, the core Promises/A+ specification does not deal with how to create, fulfill, or reject promises, choosing instead to focus on providing an interoperable then method. Future work in companion specifications may touch on these subjects.

{% endblockquote %}

Promises/A+ 规范主要是制定一个通用的回调方法 `then`，使得各个实现的版本可以形成链式结构进行回调。这使得不同的 Promise 库内部细节实现可能不一样，但是只有具有想通的 `then` 方法，返回的 Promise API 之间就可以相互调用。

下面会实现一个简单的 Promise，不想看实现的可以跳过。项目地址在[这里](https://github.com/FTAndy/mimi-promise)，欢迎更多讨论。

### Promise 构造函数，选择平台的 microtask 实现
```javascript
// Simply choose a microtask
const asyncFn = function() {
  if (typeof process === 'object' && process !== null && typeof(process.nextTick) === 'function')
    return process.nextTick
  if (typeof(setImmediate === 'function'))
    return setImmediate
  return setTimeout
}()

// States
const PENDING = 'PENDING'

const RESOLVED = 'RESOLVED'

const REJECTED = 'REJECTED'

// Constructor
function MimiPromise(executor) {
  this.state = PENDING
  this.executedData = undefined
  this.multiPromise2 = []

  resolve = (value) => {
    settlePromise(this, RESOLVED, value)
  }

  reject = (reason) => {
    settlePromise(this, REJECTED, reason)
  }

  executor(resolve, reject)
}
```

`state` 和 `executedData` 都容易理解，但是必须要理解一下为什么要维护一个 `multiPromise2` 数组。由于规范中说明，每个调用过 `then` 方法的 `promise` 对象必须返回一个新的 `promise2` 对象，所以最好的方法是当调用 `then` 方法的时候将一个属于这个 `then` 方法的 `promise2` 加入队列，在 `promise` 对象中维护这些新的 `promise2` 的状态。

- `executor`： `promise` 构造函数的执行函数参数
- `state`：`promise` 的状态
- `multiPromise2`：维护的每个注册 `then` 方法需要返回的新 `promise2`
- `resolve`：函数定义了将对象设置为 `RESOLVED` 的过程
- `reject`：函数定义了将对象设置为 `REJECTED` 的过程

最后执行构造函数 `executor`，并调用 `promise` 内部的私有方法 `resolve` 和 `reject`。

### settlePromise 如何将一个新建的 Promise settled
```javascript
function settlePromise(promise, executedState, executedData) {
  if (promise.state !== PENDING)
    return

  promise.state = executedState
  promise.executedData = executedData

  if (promise.multiPromise2.length > 0) {
    const callbackType = executedState === RESOLVED ? "resolvedCallback" : "rejectedCallback"

    for (promise2 of promise.multiPromise2) {
      asyncProcessCallback(promise, promise2, promise2[callbackType])
    }
  }
}
```

第一个判断条件很重要，因为 Promise 的状态是不可逆的。在 `settlePromise` 的过程中假如状态不是 `PENDING`，则不需要继续执行下去。

当前 `settlePromise` 的环境，可以有三种情况：

- 异步延迟执行 `settlePromise` 方法，线程已经同步注册好 `then` 方法，需要执行所有注册的 `then` 回调函数
- 同步执行 `settlePromise` 方法，`then` 方法未执行，后面需要执行的 `then` 方法会在注册的过程中直接执行
- 无论执行异步 `settlePromise` 还是同步 `settlePromise` 方法，并没有注册的 `then` 方法需要执行，只需要将本 Promise 对象的状态设置好即可

### then 方法的注册和立即执行
```javascript
MimiPromise.prototype.then = function(resolvedCallback, rejectedCallback) {
  let promise2 = new MimiPromise(() => {})

  if (typeof resolvedCallback === "function") {
      promise2.resolvedCallback = resolvedCallback;
  }
  if (typeof rejectedCallback === "function") {
      promise2.rejectedCallback = rejectedCallback;
  }

  if (this.state === PENDING) {
    this.multiPromise2.push(promise2)
  } else if (this.state === RESOLVED) {
    asyncProcessCallback(this, promise2, promise2.resolvedCallback)
  } else if (this.state === REJECTED) {
    asyncProcessCallback(this, promise2, promise2.rejectedCallback)
  }

  return promise2
}
```

每个注册 `then` 方法都需要返回一个新的 `promise2` 对象，根据当前 `promise` 对象的 state，会出现三种情况：

- 当前 `promise` 对象处于 PENDING 状态。构造函数异步执行了 `settlePromise` 方法，需要将这个 `then` 方法对应返回的 `promise2` 放入当前 `promise` 的 `multiPromise2` 队列当中，返回这个 `promise2`。以后当 `settlePromise` 方法异步执行的时候，执行全部注册的 `then` 回调方法
- 当前 `promise` 对象处于 `RESOLVED` 状态。构造函数同步执行了 `settlePromise` 方法，直接执行 `then` 注册的回调方法，返回 `promise2`。
- 当前 `promise` 对象处于 `REJECTED` 状态。构造函数同步执行了 `settlePromise` 方法，直接执行 `then` 注册的回调方法，返回 `promise2`。

### 异步执行回调函数
```javascript
function asyncProcessCallback(promise, promise2, callback) {
  asyncFn(() => {
    if (!callback) {
      settlePromise(promise2, promise.state, promise.executedData);
      return;
    }

    let x

    try {
      x = callback(promise.executedData)
    } catch (e) {
      settlePromise(promise2, REJECTED, e)
      return
    }

    settleWithX(promise2, x)
  })
}
```

这里用到我们之前选取的平台异步执行函数，异步执行 callback。假如 callback 没有定义，则将返回 `promise2` 的状态转换为当前 `promise` 的状态。然后将 callback 执行。最后再 `settleWithX` `promise2` 与 callback 返回的对象 `x`。

### 最后的 settleWithX 和 settleXthen
```javascript
function settleWithX (p, x) {
    if (x === p && x) {
        settlePromise(p, REJECTED, new TypeError("promise_circular_chain"));
        return;
    }

    var xthen, type = typeof x;
    if (x !== null && (type === "function" || type === "object")) {
        try {
            xthen = x.then;
        } catch (err) {
            settlePromise(p, REJECTED, err);
            return;
        }
        if (typeof xthen === "function") {
            settleXthen(p, x, xthen);
        } else {
            settlePromise(p, RESOLVED, x);
        }
    } else {
        settlePromise(p, RESOLVED, x);
    }
    return p;
}

function settleXthen (p, x, xthen) {
    try {
        xthen.call(x, function (y) {
            if (!x) return;
            x = null;

            settleWithX(p, y);
        }, function (r) {
            if (!x) return;
            x = null;

            settlePromise(p, REJECTED, r);
        });
    } catch (err) {
        if (x) {
            settlePromise(p, REJECTED, err);
            x = null;
        }
    }
}
```

这里的两个方法对应 Promise/A+ 规范里的第三章，由于实在太啰嗦，这里就不再过多解释了。

### 配合 async/await 使用更加美味
V8 已经原生实现了 `async/await`，Node 和各浏览器引擎的实现也会慢慢跟进，而 babel 早就加入了 `async/await`。目前客户端还是用 babel 预编译使用比较好，而 Node 需要升级到 v7 版本，并且加入 `--harmony-async-await` 参数。

Promise 其中的一个局限在于：所有操作过程都必须包含在构造函数或者 `then` 回调中执行，假如有一些变量需要累积向下链式使用，还要加入外部全局变量，或者引起回调地狱，像这样。

```javascript
let result1
let result2
let result3

getSomething1()
  .then((data) => {
    result1 = data
    // do some shit with result1
    return getSomething2()
  })
  .then((data) => {
    result2 = data
    // do some other shit with result1 and result2
    return getSomething3()
  })
  .then((data) => {
    result3 = data
    // do some other shit with result1, result2 and result3
  })
  .catch((err) => {
    console.error(err);
  })

getSomething1()
  .then((data1) => {
    // do some shit with data1
    return getSomething2()
    .then((data2) => {
      // do some shit with data1 and data2
      return getSomething3()
      .then((data3) => {
        // do some shit with data1, data2 and data3
      })
    })
  })
  .catch((err) => {
    console.error(err);
  })
```

引入了全局变量和写出了回调地狱都不是明智的做法，假如用了 `async/await`，可以这样：

```javascript
async function a() {
  try {
    const result1 = await getSomething1()
    // do some shit with result1
    const result2 = await getSomething2()
    // do some other shit with result1 and result2
    const result3 = await getSomething3()
    // do some other shit with result1, result2 and result3
  } catch (e) {
    console.error(e);
  }
}
```
`async/await` 配合 Promise，没有了 `then` 方法和回调地狱的写法是不是清爽了很多？

### 结语
本文后续其实还有更多值得挖掘的地方：

- 如何更加有效地选取平台的 microtask？
- 如何实现一个可用的符合 ECMAScript 规范的 Promise？
- microtask 和 task 在 event loop 具体的执行过程？

可以期待后续的更多内容。最后再贴一下[项目地址](https://github.com/FTAndy/mimi-promise)，欢迎继续的讨论。
### 参考资料
- [asap github](https://github.com/kriskowal/asap)
- [setImmediate github](https://github.com/YuzuJS/setImmediate)
- [yaku github](https://github.com/ysmood/yaku)
- [剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)
