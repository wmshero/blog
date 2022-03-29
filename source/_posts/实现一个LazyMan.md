---
title: 实现一个LazyMan
date: 2022-03-20 23:57:05
tags: ['javascript']
categories: 'javascript'
---

[参考](https://github.com/BetaSu/fe-hunter/issues/13)


# 要实现的功能
实现一个 LazyMan，按照以下方式调用时，得到相关输出。

## 代码示例
```js
LazyMan("Hank")
// 打印：Hi! This is Hank!

LazyMan("Hank").sleep(10).eat("dinner")
// 打印：Hi! This is Hank!
// 等待了 10 秒后
// 打印：Wake up after 10
// 打印：Eat dinner~
 
LazyMan("Hank").eat("dinner").eat("supper")
// 打印：Hi This is Hank!
// 打印：Eat dinner~
// 打印：Eat supper~
 
LazyMan("Hank").sleepFirst(5).eat("supper")
// 等待了 5 秒后
// 打印：Wake up after 5
// 打印：Hi This is Hank!
// 打印：Eat supper

LazyMan("Hank").eat("supper").sleepFirst(5)
// 等待了 5 秒后
// 打印：Wake up after 5
// 打印：Hi This is Hank!
// 打印：Eat supper
```

# 解答

```js

function LazyMan(name) {
    const { log } = console;
    const sleep = s =>
        new Promise(res =>
            setTimeout(() => log(`Wake up after ${s}`) || res(), s * 1000)
        );
    // 定义队列并切设置第一个任务
    const queue = [() => log(`Hi! This is ${name}!`)];

    // 这个里用了 push(x) && ctx 
    // push 的返回值是数组 push 后的长度 所以不会出现 0 , 可以放心在箭头函数里使用
    const ctx = {
        eat: food => queue.push(() => log(`Eat ${food}~`)) && ctx,
        sleep: s => queue.push(() => sleep(s)) && ctx,
        sleepFirst: s => queue.unshift(() => sleep(s)) && ctx
    };

    // 延迟在下一个周期执行, 为了收集执行的任务
    queueMicrotask(async () => {
        while (queue.length) {
            await queue.shift()();
        }
    });
    return ctx;
}
```


## queueMicrotask 

> 当我们期望某段代码，不阻塞当前执行的同步代码，同时又期望它尽可能快地执行时，我们就需要使用它。

在有一些框架和工具里面，如果遇到以上的这种需求，都是使用`Process.nextTick`和`Promise.resolve`来解决。
但是这些都不是很合适，比如`Promise.resolve`： 会将异常转换为一个`rejected` 的`Promise` 。`Promise.resolve`会返回一个`Promise`实例对象，而`requestMicrotask`不会，而且`requestMicrotask`语义上更加合适

### 潜在问题
由于它是一个用于指派微任务的底层 api，我们很可能会在其中无限制地指派微任务到其队列之中，这样做的效果就是，浏览器的微任务队列始终处于非空状态，这将导致控制权始终无法交还给浏览器进行下一次事件循环，然后它就卡死了。


### polyfill

`core-js` 会复杂一些，它同时考虑了 `nodejs` 和 `browser` 两种情况，同时利用链表数据结构来模拟微任务队列的执行单元，同时实现了一个 `flush` 方法表示执行全部的微任务单元。
还实现了一个 `notify` 方法，该方法会根据具体的 js 运行时环境以及 api 的支持情况，分别尝试使用 `process.nextTick`、`MutationObserver` 和  `Promise.resolve` 以及最基本的宏任务 api 来执行 flush 方法，变相模拟微任务的执行过程。
