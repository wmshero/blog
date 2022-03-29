---
title: symbol
date: 2022-03-28 23:51:31
tags: ['javascript']
categories: 'javascript'
---


# 是什么？

`symbol` 是一种基本数据类型。`Symbol()`函数会返回`symbol`类型的值，该类型具有静态属性和静态方法。它的静态属性会暴露几个内建的成员对象；它的静态方法会暴露全局的`symbol`注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法："`new Symbol()`"。

# 为什么有symbol？

每个从Symbol()返回的`symbol`值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

# 怎么使用？

## 对 symbol 使用 typeof 运算符
```js
typeof Symbol() === 'symbol'
typeof Symbol('foo') === 'symbol'
typeof Symbol.iterator === 'symbol'
```

# 坑？

