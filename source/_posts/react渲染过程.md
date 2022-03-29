---
title: react渲染过程
date: 2022-03-28 12:21:00
categories: 'react'
tag: ['react']
---

> 对于首次渲染，React 的主要工作就是将` React.render` 接收到的` VNode` 转化 `Fiber` 树，并根据 Fiber 树的层级关系，构建生成出 DOM 树并渲染至屏幕中。

> 而对于更新渲染时，`Fiber` 树已经存在于内存中了，所以 React 更关心的是计算出 `Fiber` 树中的各个节点的差异，并将变化更新到屏幕中。


react 的核心可以用`ui=fn(state)`来表示，更详细可以用
```js
const state=reconcile(update);
const UI=commit(state)
```
上面的fn可以分为如下几个部分：
+ `Scheduler` 调度器： 排序优先级，让优先级高的任务线进行reconcile
+ `Reconciler` 协调器： 找出哪些节点发生了改变，并打上不同的Flags
+ `Renderer` 渲染器： 将Reconciler中打好标签的节点渲染到视图上

那么这些模块是怎么配合工作的呢：

+ 首先`jsx`经过`babel`的`ast`词法解析后编程
    `React.createElement`：`React.createElement`函数执行后就是`jsx`对象，也称为 `virtual-dom`

+ 不管是在首次渲染还是更新状态的时候，这些渲染的任务都会经过`Scheduler`的调度，`Scheduler`会根据任务的优先级来决定将哪些任务优先进入`render`阶段.
> 比如用户触发的更新优先级非常高，如果当前正在进行一个比较耗时的任务，则这个任务就会被用户触发的更新打断，在`Scheduler`中初始化任务的时候会计算一个过期时间，不同类型的任务过期时间不同，优先级越高的任务，过期时间越短，优先级越低的任务，过期时间越长。
在最新的`Lane`模型中，则可以更加细粒度的根据二进制1的位置，来决定任务的优先级，通过二进制的融合和相交，判断任务的优先级是否足够在此次`render`的渲染。
`Scheduler`会分配一个时间片给需要渲染的任务，如果是一个非常耗时的任务，如果在一个时间片之内没有执行完成，则会从当前渲染到的Fiber节点暂停计算，让出执行权给浏览器，在之后浏览器空闲的时候从之前暂停的那个`Fiber`节点继续后面的计算，这个计算的过程就是计算`Fiber`的差异，并标记副作用。

+ 在`render`阶段：render阶段的主角是`Reconciler`，在`mount`阶段和`update`阶段，它会比较`jsx`和当前`Fiber`节点的差异（diff算法指出就是这个比较的过程），将带有副作用的`Fiber`节点标记出来，这些副作用有`Placement`（插入）、`Update`（更新）、`Deletetion`（删除）等，而这些带有副作用`Fiber`节点会加入一条`EffectList`中，在`commit`阶段就会遍历这条`EffectList`，处理相应的副作用，并且应用到真实节点上。而`Scheduler`和`Reconciler`都是在内存中工作的，所以他们不影响最后的呈现。

+ 在`commit`阶段：会遍历`EffectList`，处理相应的生命周期，将这些副作用应用到真实节点，这个过程会对应不同的渲染器，在浏览器的环境中就是`react-dom`，在`canvas`或者`svg`中就是`reac-art`等。

另外我们也可以从首次渲染和更新的时候看在`render`和`commit`这两个子阶段是如果工作的：

+ mount时：
1. 在`render`阶段会根据`jsx`对象构建新的`workInProgressFiber`树，然后将相应的`fiber`节点标记为`Placement`，表示这个`fiber`节点需要被插入到`dom`树中，然后会这些带有副作用的`fiber`节点加入一条叫做`Effect List`的链表中。

2. 在`commit`阶段会遍历`render`阶段形成的`Effect List`，执行链表上相应`fiber`节点的副作用，比如`Placement`插入，或者执行`Passive`（`useEffect`的副作用）。将这些副作用应用到真实节点上

+ update时：

在`render`阶段会根据最新状态的jsx对象对比`current Fiber`，再构建新的`workInProgressFiber`树，这个对比的过程就是diff算法，diff算法又分成单节点的对比和多节点的对比，对比的过程中同样会经历收集副作用的过程，也就是将对比出来的差异标记出来，加入Effect List中，这些对比出来的副作用例如：Placement（插入）、Update(更新)、Deletion（删除）等。
在commit阶段同样会遍历Effect List，将这些fiber节点上的副作用应用到真实节点上