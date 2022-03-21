---
title: js基础
date: 2022-03-13 15:37:16
tags: ['js']
categories: js
---

# 预编译

> **AO对象**： Activation Object ，活动性对象，执行期上下文（作用域） 

## 函数上下文
1. 寻找形参和变量声明
2. 实参值赋值给形参
3. 找函数声明，赋值
4. 执行

```js
function fn(a,b,c) {
    console.log(a); // ƒ a() { } 先找函数声明
    console.log(b); // 2
    function a() { }
    a = 1
    b = 2
    
    console.log(a); // 1
    console.log(b); // 2
    console.log(c); // undefined
}
fn(1,2)
```

## 全局上下文

> **GO对象**： Gobal Object ，全局对象，GO对象===window对象。

1. 找变量
2. 找函数声明
3. 执行

> 下面这题为什么打印出1 ？因为先找函数声明，此时a是函数，然后找赋值，此时a被覆盖为1了。

```js
var a=1;
function a{
    console.log(2)
}

console.log(a) // 1
```

下面来个综合的预编译案例：
```js
 a = 100;

function demo(e) {
    function e() { };
    arguments[0] = 2;
    console.log(e);//2
    if (a) {
        var b = 0;
    }
    var c;
    c = function sum() { } 
    a = 10;
    var a;
    console.log(b); // undefined
    f = 123;
    console.log(c); //function sum(){}
    console.log(a); // 10 
}
var a;
demo(1);
console.log(a);//100
console.log(f);// 123
```



# 作用域