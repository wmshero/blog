---
title: 深入浅出Rxjs
date: 2022-03-22 13:07:14
tags: ['Rxjs']
categories: 'Rxjs'
---

# 入门基础
## Observable 和 Observer
+ Observable "可被观察者"
+ Observer "观察者"
+ 连接两者的桥梁是 observable 对象的 函数 subscribe

RxJS中的数据流就是Observable对象，Observable实现了下⾯两种设计模式：

+ 观察者模式（Observer Pattern）
+ 迭代器模式（Iterator Pattern）

### 观察者模式

观察者模式要解决的问题，就是在一个持续产生事件的系统中，如何分割功能，让不同模块只需要处理一部分逻辑，这种分而治之的思想是基本的系统设计概念，当然，”分“很容易，关键是如何”治“。
观察者模式对“治”这个问题提的解决⽅法是这样，将逻辑分为发布者（`Publisher`）和观察者（`Observer`），其中发布者只管负责产⽣事件，它会通知所有注册挂上号的观察者，⽽不关⼼这些观察者如何处理这些事件， 相对的，观察者可以被注册上某个发布者，只管接收到事件之后就处理， ⽽不关⼼这些数据是如何产⽣的。

在RxJS的世界中，`Observable`对象是一个发布者，通过`observable`对象的`subscribe`，可以吧这个发布者和某个观察者`observe`连接起来。

在下面的代码中，`source$`是一个`Observable`对象，作为发布者，它产生的”事件“就是连续的三个整数：1，2，3

```js
import {Observable} from 'rxjs/Observable'
import ‘rxjs/add/observable/of'

const source$=Observable.of(1,2,3);
source$.subscribe(console.log);
```

扮演观察者`Observer`的是`console.log`函数，无论传入什么”事件“，它只管把”事件“输出到`console`上。

观察者模式的好处，这个模式的两方可以专心的做一件事，而且可以任意的组合，也就是，复杂的问题被分成三个小问题：

1. 如何产生事件，这是发布者的责任，在Rxjs中是Observable对象的工作。
2. 如何响应事件，这是观察者的责任，在Rxjs中由subscribe的参数来决定。
3. 什么样的发布者关联什么样的观察者，也就是何时调用subscribe。

### 迭代器模式

迭代器 指的是能够遍历一个数据集合的对象。也可称为”游标“，就像是一个移动的指针一样，从集合中的一个元素移到另一个元素，最后完成整个集合的遍历。

这个设计模式通常包含以下几个函数：

+ getCurrent,获取当前被游标所指向的元素。
+ moveToNext，将游标移动到下一个元素，调用这个函数之后，getCurrent获得的元素就会不同。
+ isDone，判断是否已经遍历完了所有的元素。

示例：
```js
const iterator=getIterator()
while(iterator.isDone())
{
  console.log(iterator.getCurrent())
	iterator.moveToNext()
}
```

在编程的世界中，所谓“拉”（`pull`）或者“推”（`push`），都是从数据消费者⾓度的描述，⽐如，在⽹页应⽤中，如果是⽹页主动通过AJAX请求从服务器获取数据，这是“拉”，如果⽹页和服务器建⽴了`websocket`通道，然后，不需要⽹页主动请求，服务器都可以通过websocket通道推送数据到 ⽹页中，这是“推”。在RxJS中，作为迭代器的使⽤者，并不需要主动去从Observable 中“拉”数据，⽽是只要subscribe上Observable对象之后，⾃然就能够收到消息的推送，这就是观察者模式和迭代器两种模式结合的强⼤之处。

#### 创建Observable
每个Observable对象，代表的就是一段时间范围内发生的一系列事件
Rxjs结合了观察者模式和迭代器模式，故： `Observable=publish+iterator`
下面的代码创造和使用了一个简单的Observable对象：

```js
import {Observable} from 'rxjs/Observable'
const onSubscribe=observer=>{
	observer.next(1);
  observer.next(2);
  observer.next(3);
};
const source$=new Observable(onSubscribe)
const theObserver={
	next:item=>console.log(item)
}
source$.subscribe(theObserver);

//输出：
1
2
3
```

> 分析：
1. 导入了Observable类
2. 创造了函数onSubscribe,这个函数会被用作Observable构造函数的参数，这个函数参数完全决定了Observable对象的行为。onSubscribe函数接受一个名叫observer的参数，函数体内，调用参数observer的next函数，把数据”推“给observer。
3. 调⽤Observable构造函数，产⽣⼀个名为source$的数据流对象。
4. 创造观察者theObserver。
5. 通过subscribe函数将theObserver和source$关联起来

创建Observable对象也就是创建⼀个“发布者”，⼀个“观察者”调⽤某个Observable对象的subscribe函数，对应的`onSubscribe`函数就会被调⽤，参数就是“观察者”对象，`onSubscribe`函数中可以任意操作“观察者”对象。这个过程，就等于在这个Observable对象上挂了号，以后当这个Observable对象产⽣数据时，观察者就会获得通知。
在上⾯的代码中，“观察者”就是`theObserver`。在RxJS中，Observable是⼀个特殊类，它接受⼀个处理Observer的函数，⽽Observer就是⼀个普通的对象，没有什么神奇之处，对Observer对象的要求只有它必须包含⼀个名为`next`的属性，这个属性的值是⼀个函数， ⽤于接收被“推”过来的数据。

### 跨域时间的Observable

Observer是被“推”数据的，在执⾏过程中处于被动地位，所以，异步操作，还是应该交给Observable来做，Observable既然能够“推”数据，那同时负责推送数据的节奏，天经地义，完全合理。
```js
const onSubscribe=observer=>{
		let number=1
    const handle=setInterval(()=>{
    	observer.next(number++);
      if(number>3){
      	clearInterval(handle);
      }
    },1000)

}
```

### 永无止境的Observable

Observable可以产⽣**⽆限多**的数据。
Observable对象调用观察者`next`传递数据的动作，可以将产生动作的行为称为”吐出“
Observable对象每吐出一个数据，然后数据会被Observe消化处理掉，不会造成数据的堆积。现实中有很多数据流就是永无止境的，例如点击事件。但也有一些数据流是会终止的，那么如何终止数据流呢？
Observable会停止吐出数据，停止调用next函数推送数据，但是不会给Observe一个终止信号，Observe还在傻傻的等着接收Observable传来的数据，所以这样子还是不行，得需要一个宣称Observable终结的方式。

### Observable的终结
此处需要使用到Observe的`complete`函数：

```js
const theObserve={
	next:item=>console.log(item)
  complete:()=>console.log('no more data')
}
```

修改onSubscribe函数

```js
const onSubscribe=observer=>{
		let number=1
    const handle=setInterval(()=>{
    	observer.next(number++);
      if(number>3){
      	clearInterval(handle);
        observer.complete()//调用Observe的complete函数
      }
    },1000)

}
```

Observe对象的complete函数何时调用，是有Observable对象决定的，所以完结信号是Observable”推“给Observe的。

### Observable的出错处理

在Observable和Observer的交流渠道中增加一个新的函数`error`

```js
import {Observable} from 'rxjs/Observable'
const onSubscribe=observer=>{
	observer.next(1);
  observer.error('something wrong');
  observer.complete();
};
const source$=new Observable(onSubscribe)
const theObserve={
	next:item=>console.log(item)
  error:err=>console.log(err)
  complete:()=>console.log('no more data')
}
source$.subscribe(theObserver);
```
以上输出结果为：

```js
1 
something wrong
```

**在Rxjs中，一个Observable对象只能有一个终结方式，要么出错，要么完结**

onSubscribe的参数observer并不是传给subscribe的参数theObserver，⽽是对theObserver的包装，所以，即使在`observer.error`被调⽤之后强⾏调⽤`observer.complete`，也不会真正调⽤到theObserver的complete函数。

### Observe的简单形式

```js
source$.subscribe(
 item=>console.log(item),
 err=>console.log(err),
 ()=>console.log('no more data')
 );
```
如果不处理异常行为，可以将第二个参数改为null。

### 退订Observable
onSubscribe函数返回一个对象，对象包含`unsubscribe`函数，这个函数清除了`setInterval`产生的效果。
```js
const onSubscribe = observer => { 
  let number = 1; 
  const handle = setInterval(() => { 
  	observer.next(number++); 
  }, 1000); 
  return { 
  	unsubscribe: () => { 
 			 clearInterval(handle); 
 		 } 
  }; 
};
const source$=new Observable(onSubscribe)
const subscription=source$.subscribe(item=>console.log(item))
setTimeout(()=>{
	subscription.unsubscribe();
},3500)
```

虽然unsubscribe函数调⽤之后，作为Observer不再接受到被推送的数据，但是作为Observable的source$并没有终结，因为始终没有调⽤`complete`，只不过它再也不会调⽤next函数了。
**Observable产⽣的事件，只有Observer通过subscribe订阅之后才会收到，在unsubscribe之后就不会再收到。**

### Hot Observable 和 Cold Observable
假设有这样的场景，⼀个Observable对象有两个Observer对象来订阅，⽽且这两个Observer对象并不是同时订阅，第⼀个Observer对象订阅N秒钟之后，第⼆个Observer对象才订阅同⼀个Observable对象，⽽且，在这N秒钟之内，Observable对象已经吐出了⼀些数据。现在问题来了，后订阅上的Observer，是不是应该接收到“错过”的那些数据呢？

* 选择A：错过就错过了，只需要接受从订阅那⼀刻开始Observable产⽣的数据就⾏。
* 选择B：不能错过，需要获取Observable之前产⽣的数据。

选择A，称这样的Observable为`Hot Observable`
选择B，称之为`ColdObservable。`

### Hot Observable
对于⼀个HotObservable，概念上是有⼀个独⽴于Observable对象的“⽣产者”，这个“⽣产者”的创建和subscribe调⽤没有关系，subscribe调⽤只是让Observer连接上“⽣产者”⽽已，

```js
const producer=new producer()
const hot$=new Observable(observer=>{
	//然后让observer去接收producer产生的数据
})
```

### Cold Observable

如果设想有⼀个数据“⽣产者”（producer）的⾓⾊，那么，对于Cold Observable，每⼀次订阅，都会产⽣⼀个新的“⽣产者”。

```js
const cold$=new Observable(observer=>{
  const producer=new producer()
	//然后让observer去接收producer产生的数据
})
```

# 操作符

对于现实中复杂的问题，并不会创造⼀个数据流之后就直接通过subscribe接上⼀个Observer，往往需要对这个数据流做⼀系列处理，然后才交给Observer。就像⼀个管道，数据从管道的⼀段流⼊，途径管道各个环节，当数据到达Observer的时候，已经被管道操作过，有的数据已经被中途过滤抛弃掉了，有的数据已经被改变了原来的形态，⽽且最后的数据可能来⾃多个数据源，最后Observer只需要处理能够⾛到终点的数据。

> 在RxJS中，有⼀系列⽤于产⽣Observable函数，这些函数有的凭空创造Observable对象，有的根据外部数据源产⽣Observable对象，更多的是根据其他的Observable中的数据来产⽣新的Observable对象，也就是把上游数据转化为下游数据，所有这些函数统称为操作符。

```js
import {Observable} from 'rxjs/Observable'
import 'rxjs/add/operator/map'

const onSubscribe=observer=>{
	observer.next(1);
 	observer.next(2);
	observer.next(3);
};
const source$=Observable.create(onSubscribe)
source$.map(x=>x*x).subscribe(console.log);
```

**create可以创造Observable对象,每⼀个操作符都是创造⼀个新的Observable对象，不会对上游的Observable对象做任何修改，符合函数式编程的”数据不可变“要求,操作符就是⽤来产⽣全新Observable对象的函数**

操作符
### 操作符分类
* 功能分类
* 创建类
* 转化类
* 过滤类
* 合并类
* 多播类
* 错误处理类
* 辅助工具类
* 条件分支类
* 数字和合计类
* 背压控制类
* 可连接类
* 高阶Observable处理类
* 静态和实例分类

所有的操作符都是函数，不过有的操作符是Observable类的静态函数，也就是不需要Observable实例就可以执⾏的函数，所以称为“**静态操作符**”；

例如of:
`Observable.of=functionToImplementOf;`
在使用方式上：
`const source$=Observable.of(/*一些参数*/);`

另⼀类操作符是Observable的实例函数，前提是要有⼀个创建好的Observable对象，这⼀类称为“**实例操作符**”。 比如`map`

```js
Observable.prototype.map=functionToImplementMap;
const result$ = source$.map(/*一些参数*/);
```

## 创建类操作符

创建类操作符可以凭空创建Observable，并且不需要从其他Observable中获取数据，所以在数据管道中，创建类操作符是数据流的源头。
创建同步数据流
也可以称为同步Observable，需要关心的是：
产生哪些数据
数据之间的先后顺序如何
`create`

```js
Observable.create=function(subscribe){
	return new Observable(subscribe)
}
```

## of 列举数据

利用of操作符可以创建指定数据集合的Observable对象。
```js
import {Observable} from 'rxjs/Observable'
import 'rxjs/add/Observable/of'

const source$=Observable.of(1,2,3)

///////////或者///////////////

import {of} from 'rxjs/observable/of'
const source$=of(1,2,3)

source$.subscribe(
	console.log,
  null,
  ()=>console.log('complete')
)
```
of吐出的数据 1，2，3是同步输出的，没有时间间隔，of 产生的Cold Observable,对于每一个Observer都会重复吐出同样的一组数据，所以可以反复使用。

## range 指定范围

产生一个从1到100所有正整数的Observable对象：

`const source$=Observable.range(1,100)`
range也是同步的方式，一次性将100个数据都推给Observe

## generate 循环创建
generate类似于for循环
```js
const source$ = Observable.generate( 
  2, // 初始值，相当于for循环中的i=2 
  value => value < 10, //继续的条件，相当于for中的条件判断
  value => value + 2, //每次值的递增 
  value => value * value // 产⽣的结果 
);
```

## repeat 重复数据的数据流
repeat 的 功能是可以重复上游Observable中的数据若干次。
```js
const source$=Observable.of(1,2,3)
const repeated$=source$.repeat(10)
```

`source$`产生的Observable对象会产生1、2、3三个数据，这个Observable对象是`repeated$`的上游，理由repeat操作符，重复source$内容10遍。
repeat重复功能是基于上游的完结时机的，如果上游没有完结，那么使用repeat是完全没有意义的。

## empty 、 never 、 throw
### empty
empty产生一个直接完结的Observable对象，没有参数，不产生任何数据。
`const source$=Observable.empty()`
### throw
throw产生的Observable对象也是什么都不做，直接出错，抛出错误就是throw的参数。
`const source$=Observable.throw(New Error('Oops'))`
### never
什么也不做，不吐出数据，不完结，不产生错误，就这么待着，到永远。
`const source$=Observable.never()`

## 创建异步数据的Observable对象

interval和timer ：定时产生数据
### interval
等同于 JS 中的 setInterval和setTimeout
**interval 接收 一个数值类型的参数， 代表产生数据的间隔毫秒数， 返回的Observable 对象 是按照这个时间间隔 输出递增的整数序列。**
`const source$=Observable.interval(1000)`

interval是不会主动调用下游的complete，要想停止这个数据序列，需要退订。
### timer
timer的第⼀个参数可以是⼀个数值，也可以是⼀个Date类型的对象。如果第⼀个参数是数值，代表毫秒数，产⽣的Observable对象在指定毫秒之后会吐出⼀个数据0，然后⽴刻完结。
`const source$=Observable.timer(1000)`
上⾯的功能也可以通过传递⼀个Date对象给timer来实现，代码如下

```js
const now=newDate();
const later=newDate(now.getTime()+1000);
const source$=Observable.timer(later);
```

使⽤数值参数还是使⽤Date对象作为参数，应该根据具体情况确定，如果明确延时产⽣数据的时间间隔，那就应该⽤数值作为参数，如果明确的是⼀个时间点，那⽤Date对象毫⽆疑问是最佳选择。
timer还⽀持第⼆个参数，如果使⽤第⼆个参数，那就会产⽣⼀个持续吐出数据的Observable对象，类似interval的数据流。第⼆个参数指定的是各数据之间的时间间隔，从被订阅到产⽣第⼀个数据0的时间间隔，依然由第⼀个参数决定。
在下⾯的⽰例代码中，source$被订阅之后，2秒钟的时刻吐出0，然后3秒钟的时刻吐出1，4秒钟的时刻吐出2……依次类推：

`const source$=Observable.timer(2000,1000);`

### from : 可把一切转为Observable
from可能是创建类操作符中包容性最强的⼀个了，因为它接受的参数只要“像”Observable就⾏，然后根据参数中的数据产⽣⼀个真正的 Observable对象。
`const source$=Observable.from([1,2,3])`
from 也可以将generator函数的结果转为Observable对象
```js
function * generateNumber(max){
	for(let i=1;i<=max;++i){
  	yield i;
  }
}

const source$=Observable.from(generateNumber(3))
```
结果为 1，2，3
### from 也可以接收字符串作为参数
`const source$=Observable.from('abc')`
from会将参数当做是Iterable看待，字符串abc在from看来就是数组['a','b','c']

### fromPromise :异步处理的交接
 如果from的参数是Promise对象，那么这个Promise成功结束，from产⽣的Observable对象就会吐出Promise成功的结果，并且⽴刻结束，⽰例代码如下：

```js
const promise=Promise.resolve('good');
const source$=Observable.from(promise);

source$.subscribe( 
  console.log,
	error=>console.log('catch',error),
  ()=>console.log('complete')
 )
```
Promise对象虽然也⽀持异步操作，但是它只有⼀个结果，所以当 Promise成功完成的时候，from也知道不会再有新的数据了，所以⽴刻完结了产⽣的Observable对象。
当Promise对象以失败⽽告终的时候，from产⽣的Observable对象也会⽴刻产⽣失败事件。

### fromEvent
fromEvent的第一个参数是一个事件源，在浏览器中，最常见的事件源是DOM元素，第二个参数是事件的名称，对应的DOM事件就是click、mousemove这样的字符串。
```js
<button id="clickMe">ClickMe</button>
<div  id="text">0</div>
```

目的：点击按钮，id为text的div数字增1。

```js
let clickCount=0;
const event$=Rx.Observable.fromEvent(document.querySelector('#clickMe'),'click');
event$.subscribe(
	()=>{
  	document.querySelector('#text').innerText=++clickCount
  }  
)
```

**fromEvent是DOM和RxJS世界的桥梁**，产⽣Observable对象之后，就可以完全按照RxJS的规则来处理数据
fromEvent除了可以从DOM中获得数据，还可以从Node.js的events中获得数据
fromEvent产⽣的是`HotObservable`