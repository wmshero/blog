---
title: react fiber
date: 2022-03-06 12:11:17
categories: 'react'
tag: ['react','fiber']
toc: true
---


# react 15 渲染方式
> 缺点： 如果界面节点多，层次深，递归渲染比较耗时，JS是单线程的，而且UI线程和JS线程是互斥的，页面会出现卡顿的现象。

```js
let element = (<div id='0' className='red'>
  <div id='1'>1</div>
  <div id='2'>2</div>
</div>)


function render(element, rootParent) {
  // console.log(JSON.stringify(element, null, 2)) 
  let dom = document.createElement(element.type)
  Object.keys(element.props).forEach(v => {
    if(v==='children'){
      if(Array.isArray(element.props[v])){
        element.props[v].forEach(childrenProp=>{
          render(childrenProp,dom)
        })
      }else{
        dom.innerHTML=element.props[v]
      }
    }else{
      dom[v] = element.props[v]
    }
  })
  rootParent.appendChild(dom)
}

render(element, document.getElementById('root'))

```

# 什么是fiber 

1.   fiber 是一种 ** 数据结构**，它可以使用一个纯JS对象来表示

```js
const fiber={
    stateNode, //节点实例
    child,     //子节点
    sibling,   //兄弟节点
    return,    //父节点
}
```

2. fiber 是一个执行单元，每次执行完一个执行单元，react就会检查现在还剩多少时间，如果没有时间就将控制权让出去。


3. fiber关键特性

* 增量渲染 （将渲染任务进行拆分，均匀到每一帧去执行）
* 可暂停、终止，复用渲染任务
* 不同更新的优先级
* 并发方面新的基础能力


# fiber 运行流程

## 帧的概念
## window.requestAnimationFrame
[window.requestAnimationFrame()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame) 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行

> 注意：若你想在浏览器下次重绘之前继续更新下一帧动画，那么回调函数自身必须再次调用window.requestAnimationFrame()

```js 
window.requestAnimationFrame(callback);
```

### 范例

```js
const element = document.getElementById('some-element-you-want-to-animate');
let start;

function step(timestamp) {
  if (start === undefined)
    start = timestamp;
  const elapsed = timestamp - start;

  //这里使用`Math.min()`确保元素刚好停在200px的位置。
  element.style.transform = 'translateX(' + Math.min(0.1 * elapsed, 200) + 'px)';

  if (elapsed < 2000) { // 在两秒后停止动画
    window.requestAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```


## requestIdleCallback

> 这是一个实验中的功能,不是所有的浏览器豆兼容。 react中是模拟了一个类似`requestIdleCallback`的功能

[window.requestIdleCallback()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 方法插入一个函数，这个函数将在浏览器**空闲**时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。

你可以在空闲回调函数中调用`requestIdleCallback()`，以便在下一次通过事件循环之前调度另一个回调。

### 语法

```js
var handle = window.requestIdleCallback(callback[, options])
```

### 返回值
一个ID，可以把它传入 `Window.cancelIdleCallback()` 方法来结束回调。


## MessageChannel

Channel Messaging API的`MessageChannel` 接口允许我们创建一个新的**消息通道**，并通过它的两个`MessagePort` 属性发送数据.

> 此特性在 Web Worker 中可用

### 示例

在以下代码块中，您可以看到使用`MessageChannel`构造函数实例化了一个`channel`对象。当`iframe`加载完毕，我们使用`MessagePort.postMessage`方法把一条消息和`MessageChannel.port2`传递给`iframe`。`handleMessage`处理程序将会从`iframe`中（使用`MessagePort.onmessage`监听事件）接收到信息，将数据其放入`innerHTML`中。

```js
let channel = new MessageChannel();
let para = document.querySelector('p');

let ifr = document.querySelector('iframe');
let otherWindow = ifr.contentWindow;

ifr.addEventListener("load", iframeLoaded, false);

function iframeLoaded() {
  otherWindow.postMessage('Hello from the main page!', '*', [channel.port2]);
}

channel.port1.onmessage = handleMessage;
function handleMessage(e) {
  para.innerHTML = e.data;
}   

```

* 目前` requestIdleCallback`只有chrome支持
* 所以React 利用 MessageChannel模拟了RequestIdleCallback，将毁掉延迟到绘制操作之后执行。
* MessageChannel API 允许我们创建一个新的消息通道，并通过他的两个MessagePort属性发送数据。
* MessageChannel 创建了一个通信的管道，这个管道又两个端口，每个端口都可以用过postMessage发送数据，而应该端口只要绑定了onMessage回调方法，就可以接受另外一个端口传过来的数据。
* MessageChannel是一个宏任务。

### 示例
```js
let channel=new MessageChannel()
let port1=channel.port1
let port2=channel.port2
port1.onmessage=function(e){
    console.log('port1 receive data:',e.data);
}
port2.onmessage=function(e){
    console.log('port2 receive data:',e.data);
}
port1.postMessage("port1数据")
port2.postMessage("port2数据")


//打印结果
/* 
port2 receive data: port1数据
port1 receive data: port2数据
 */
```

## fiber 执行阶段
 每次渲染有两个阶段： `Reconciliation`(协调render阶段)和`Commit`（提交阶段）
 
* 协调的阶段： 可以认为是`Diff`阶段，这个阶段可以被终止，这个阶段会找出所有的节点变更，例如节点新增、删除、属性变更等等，这些变更称为副作用。
* 提交阶段： 将上一阶段计算出来的需要处理的副作用（effects）一次性执行了。这个阶段必须同步执行，不能被打断。

### 遍历规则

按照深度优先：
1. 下一个节点：先儿子，后弟弟，再叔叔
2. 自己的所有子节点完成后自己完成

# 深度剖析fiber单元处理过程以及EffectList的构建过程

React 框架内部的运作可以分为3层：
* virtual DOM 层，描述页面长什么样
* Reconciler 层，负责调用组件生命周期方法，进行Diff算法等。
* Renderer 层，根据不同的平台，渲染出相应的页面. 

为了加以区分，以前的 Reconciler 被命名为`Stack Reconciler`。Stack Reconciler 运作的过程是不能被打断的，必须一条道走到黑,而 `Fiber Reconciler` 每执行一段时间，都会将控制权交回给浏览器，可以`分段执行`。
为了达到这种效果，就需要有一个调度器 (`Scheduler`) 来进行任务分配。任务的优先级有六种:

* `synchronous`，与之前的Stack Reconciler操作一样，同步执行
* `task`，在next tick之前执行
* `animation`，下一帧之前执行
* `high`，在不久的将来立即执行
* `low`，稍微延迟执行也没关系
* `offscreen`，下一次render时或scroll时才执行


Fiber Reconciler 在执行过程中，会分为 2 个阶段。
![阶段图](https://segmentfault.com/img/bVboJH6?w=1076&h=697)
1. 阶段一，生成 Fiber 树，得出需要更新的节点信息。这一步是一个渐进的过程，可以被打断。
2. 阶段二，将需要更新的节点一次过批量更新，这个过程不能被打断。

## fiber树

Fiber Reconciler 在阶段一进行 Diff 计算的时候，会生成一棵 Fiber 树。这棵树是在 Virtual DOM 树的基础上增加额外的信息来生成的，它本质来说是一个链表。
![])(https://segmentfault.com/img/bVboJHa?w=970&h=732)
![])(https://segmentfault.com/img/bVboJHa?w=970&h=732)

Fiber 树在首次渲染的时候会一次过生成。在后续需要 Diff 的时候，会根据已有树和最新 Virtual DOM 的信息，生成一棵新的树。这颗新树每生成一个新的节点，都会将控制权交回给主线程，去检查有没有优先级更高的任务需要执行。如果没有，则继续构建树的过程：
![](https://segmentfault.com/img/bVboJNB?w=872&h=785)

如果过程中有优先级更高的任务需要进行，则 `Fiber Reconciler` 会丢弃正在生成的树，在空闲的时候再重新执行一遍。

在构造 Fiber 树的过程中，Fiber Reconciler 会将需要更新的节点信息保存在`Effect List`当中，在阶段二执行的时候，会批量更新相应的节点。



## 代码实现
```js
let root = document.getElementById("root");
// 下一个工作单元
// fiber 其实也是一个普通的JS对象
let workInProgressRoot = {
    stateNode: root, // 此fiber对应的DOM节点
    props: {
        children:[element]
    }
}
let nextUnitOfWork = workInProgressRoot;
const  PLACEMENT = "PLACEMENT"
// 定义一个工作循环
function workloop(deadline) { 
    console.log("开始工作循环");
    while (nextUnitOfWork&&deadline.timeRemaining()>0) { 
        nextUnitOfWork =  performUnitOfWork(nextUnitOfWork);
    }
    if (!nextUnitOfWork) { 
        commitRoot();
    }
}
function commitRoot() { 
    let currentFiber = workInProgressRoot.firstEffect;
    while (currentFiber) { 
        console.log("commitRoot:", currentFiber.props.id);
        if (currentFiber.effectTag === "PLACEMENT") { 
            currentFiber.return.stateNode.appendChild(currentFiber.stateNode)
        }
        currentFiber = currentFiber.nextEffect;
    }
    workInProgressRoot = null;
}
/**
 * beginWork 1. 创建此Fiber的真实DOM
 * 通过虚拟DOM创建Fiber树结构
 * @param {*} workingInProgressFiber 
 */
function performUnitOfWork(workingInProgressFiber) { 
    beginWork(workingInProgressFiber);
    if (workingInProgressFiber.child) { 
        return workingInProgressFiber.child;
    }
    while (workingInProgressFiber) { 
        // 如果没有儿子当前节点其实就结束了
        completeUnitOfWork(workingInProgressFiber);
        if (workingInProgressFiber.sibling) { 
            return workingInProgressFiber.sibling;
        }
        workingInProgressFiber = workingInProgressFiber.return;
    }
}
function completeUnitOfWork(workingInProgressFiber) { 
    console.log("completeUnitOfWork", workingInProgressFiber.props.id);
    // 构建副作用链，上面只有副作用的节点
    let returnFiber = workingInProgressFiber.return;// A1
    if (returnFiber) { 
        // 把当前fiber有副作用的子链表挂载到父身上
        if (!returnFiber.firstEffect) { 
            returnFiber.firstEffect = workingInProgressFiber.firstEffect;
        }
        if (workingInProgressFiber.lastEffect) { 
            if (returnFiber.lastEffect) { 
                returnFiber.lastEffect.nextEffect = workingInProgressFiber.firstEffect;
            }
            returnFiber.lastEffect = workingInProgressFiber.lastEffect;
        }
        if (workingInProgressFiber.effectTag) { 
            if (returnFiber.lastEffect) {
                returnFiber.lastEffect.nextEffect = workingInProgressFiber;
            } else { 
                returnFiber.firstEffect = workingInProgressFiber;
            }
            returnFiber.lastEffect = workingInProgressFiber;
        }
    }



}
function beginWork(workingInProgressFiber) { 
    console.log("beginWork", workingInProgressFiber.props.id);
    if (!workingInProgressFiber.stateNode) { 
      workingInProgressFiber.stateNode =   document.createElement(workingInProgressFiber.type);
    }
    for (let key in workingInProgressFiber.props) { 
        if (key !== "children")
            workingInProgressFiber.stateNode[key] = workingInProgressFiber.props[key];
    }// 在beginwork 里面是不挂载的
    // 创建子Fiber
    let previousFiber;
    Array.isArray(workingInProgressFiber.props.children)&&workingInProgressFiber.props.children.forEach((child,index) => { 
        let childFiber = {
            type: child.type,// DOM节点类型
            props: child.props,
            return: workingInProgressFiber,
            effectTag:PLACEMENT,// 这个fiber必须要插入到父节点中
            nextEffect: null,// 下一个副作用节点
        }
        if (index === 0) {
            workingInProgressFiber.child = childFiber;
        } else { 
            previousFiber.sibling = childFiber;
        }
        previousFiber = childFiber;
    })
}
// 空闲时间
requestIdleCallback(workloop);
```


##  reconcile调和

> 替换、更新、删除节点，可在fiber上打上tag，例如`fiber.effectTag='REPLACEMENT|DELETION|UPDATE'`

* 新老节点类型一样，复用老节点dom,更新props即可
* 如果类型不一样，而且新的节点存在，创建新的节点替换老节点
* 如果类型不一样，没有新节点，有老节点，删除老节点