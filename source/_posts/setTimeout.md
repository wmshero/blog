---
title: setTimeout
date: 2022-03-13 21:27:58
tags: ['javascript','setTimeout']
categories: javascript
---

小试一题： 0-99的乱序输出，如何改成顺序输出

```js
        function print(n) {
            setTimeout(() => {
                console.log(n);
                return () => { }
            }, Math.floor(Math.random() * 1000))
        }
        for (var i = 0; i < 100; i++) {
            print(i)
        }
```
1.立即执行函数
```js
        function print(n) {
            setTimeout((() => {
                console.log(n);
                return () => { }
            })(), Math.floor(Math.random() * 1000))
        }
        for (var i = 0; i < 100; i++) {
            print(i)
        }
```
2. setTimout的第二个参数

```js
        function print(n) {
            setTimeout((() => {
                console.log(n);
                return () => { }
            }).call(null,[]), Math.floor(Math.random() * 1000))
        }
        for (var i = 0; i < 100; i++) {
            print(i)
        }
```

3. setTimeout的第三个参数
```js
  function print(n) {
      setTimeout(() => {
          console.log(n);
          return () => { }
      }),10, Math.floor(Math.random() * 1000)
  }
  for (var i = 0; i < 100; i++) {
      print(i)
  }

```