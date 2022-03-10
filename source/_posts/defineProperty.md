---
title: Object.defineProperty
date: 2022-03-10 23:00:12
tags: ['js']
categories: 'js'
---


`Object.defineProperty(obj, 'key', descriptor);`

其中 `descriptor` 可拥有的键值

* `configurable` 表示对象的属性是否可以被删除，以及除 value 和 writable 特性外的其他特性是否可以被修改。
* `enumerable`  是否可以在 for...in 循环和 Object.keys() 中被枚举。
* `value` 该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。
* `writable` 可写的
* `get`
* `set`

# 1. 使 `a === 1 && a === 2 && a === 3` 为true
```js
var _default=0
Object.defineProperty(window, 'a', {
    get(){
        return ++_default
    }
})


if (a === 1 && a === 2 && a === 3) {
    console.log('a',a); 
    console.log('相等哦');
}
```

# 2. console.log(_+_+_+_+) 输出 abc...z 

利用 `Object.defineProperty`和`ASCII`码
```js
Object.defineProperty(window, '_', {
        get() {
            this._c = this._c || 'a'.charCodeAt(0)
            this._ch = String.fromCharCode(this._c)
            if (this._c >= 'a'.charCodeAt(0) + 26) return
            ++this._c
            return this._ch
        }
    })


    console.log(_ + _ + _ + _ + _); //abcde

```


# 3. 使   `{a: 1, b: 2, c: 3 }`变成` { a: 3, b: 3, c: 5 }`

```js
  const obj = {
            a: 1, b: 2, c: 3
        }

        for (var k in obj) {
            ++obj[k]
            Object.defineProperty(obj, k, {
                writable: true,
                configurable: true,
                enumerable: true,
                value: k === 'b' ? obj[k] : ++obj[k]
            })
        }
        console.log(obj);
        /*
        {
            a:3,
            b:3,
            c:5
        }
        */
```