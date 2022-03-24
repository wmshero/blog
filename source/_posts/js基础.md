---
title: js基础
date: 2022-03-13 15:37:16
tags: ['javascript']
categories: 'javascript'
---

[思维导图](https://www.yuque.com/u2417328/wm/atd0ic)
[其他地方](https://1494601749.gitbook.io/wmspace/)

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


1. 除了函数外，js是没有块级作用域。
2. 作用域链：内部可以访问外部的变量，但是外部不能访问内部的变量。
	 注意：如果内部有，优先查找到内部，如果内部没有就查找外部的。
3. 注意声明变量是用var还是没有写（window.）
4. 注意：js有变量提升的机制【变量悬挂声明】
5. 优先级：声明变量 > 声明普通函数 > 参数 > 变量提升

```js
function c() {
  var b = 1;
  function a() {
    console.log(b); //undefined
    var b = 2;
    console.log(b); //2
  }
  a();
  console.log(b); //1
}
c();
```

```js
var name = "a";
(function () {
  if (typeof name == "undefined") {
    var name = "b";
    console.log("111" + name);
  } else {
    console.log("222" + name);
  }
})();
// "a" -> AO里面变量提升 'undefined'-> "b"

//111b
```


```js
function fun(a) {
  var a = 10;
  function a() {}
  console.log(a);
}
fun(100);

// 10   100 -> function a(){}-> 10

```