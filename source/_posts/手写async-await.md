---
title: 手写async await
date: 2022-02-20 16:37:13
categories: 'javascript'
tag: ['javascript']
---

# 前言

有人说`async `函数是`generator `函数的语法糖，接下来使用generator函数实现async。

# 示例

```javascript
const getData = () => new Promise(resolve => setTimeout(() => resolve("data"), 1000))

async function test() {
    const data = await getData()
    console.log('data',data);
    return 'success'
}

test().then(res => console.log(res))
```

> 以上代码使用generator函数表示如下：

```javascript
function* testG() {
    const data = yield getData()
    console.log('data',data);
    return 'success'
}
```

> 但是generator函数不会自动执行，每次调用它的next方法，会停留在下一个yield的位置。所以，我们需要编写一个自动执行函数

```js
var test = asyncToGenerator(
    function* testG() {
        const data = yield getData()
        console.log('data',data);
        return 'success'
    }
)


test().then(res => console.log(res))
```

asyncToGenerator接受一个generator函数，返回一个promise

```js
function* testG() {
  // await被编译成了yield
  const data = yield getData()
  console.log('data: ', data);
  const data2 = yield getData()
  console.log('data2: ', data2);
  return 'success'
}

var gen = testG()

var dataPromise = gen.next()

dataPromise.then((value1) => {
    // data1的value被拿到了 继续调用next并且传递给data
    var data2Promise = gen.next(value1)
    
    // console.log('data: ', data);
    // 此时就会打印出data
    
    data2Promise.value.then((value2) => {
        // data2的value拿到了 继续调用next并且传递value2
         gen.next(value2)
         
        // console.log('data2: ', data2);
        // 此时就会打印出data2
    })
})

```

# 实现

```js

function asyncToGenerator(generatorFunc) {
    return function () {
        const gen = generatorFunc.apply(this, arguments)
        return new Promise((resolve, reject) => {
            function step(key, arg) {
                let generatorResult
                try {
                    generatorResult = gen[key](arg)
                } catch (err) {
                    return reject(err)
                }
                const { value, done } = generatorResult
                if (done) {
                    return resolve(value)
                } else {
                    return Promise.resolve(value).then(val => step('next'), val), err => step('throw', err)
                }
            }
            step("next")
        })
    }
}
```

# 完整代码

```js
/**
 * async的执行原理
 * 其实就是自动执行generator函数
 * 暂时不考虑genertor的编译步骤（更复杂）
 */

const getData = () =>
  new Promise(resolve => setTimeout(() => resolve("data"), 1000))

// 这样的一个async函数 应该再1秒后打印data
async function test() {
  const data = await getData()

  console.log(data)
  return data
}

// async函数会被编译成generator函数 (babel会编译成更本质的形态，这里我们直接用generator)
function* testG() {
  // await被编译成了yield
  const data = yield getData()
  console.log('data: ', data);
  const data2 = yield getData()
  console.log('data2: ', data2);
  return data + '123'
}

function asyncToGenerator(generatorFunc) {
  return function() {
    const gen = generatorFunc.apply(this, arguments)

    return new Promise((resolve, reject) => {
      function step(key, arg) {
        let generatorResult
        try {
          generatorResult = gen[key](arg)
        } catch (error) {
          return reject(error)
        }

        const { value, done } = generatorResult

        if (done) {
          return resolve(value)
        } else {
          return Promise.resolve(value).then(
            function onResolve(val) {
              step("next", val)
            },
            function onReject(err) {
              step("throw", err)
            },
          )
        }
      }
      step("next")
    })
  }
}

const testGAsync = asyncToGenerator(testG)
testGAsync().then(result => {
  console.log(result)
})
```



