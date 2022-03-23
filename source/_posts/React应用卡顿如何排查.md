---
title: React应用卡顿如何排查
date: 2022-03-23 15:22:42
tags: ['性能优化','React']
categories: '性能优化'
---

> 排查组件性能瓶颈，重点放在React上，而不是通用的性能优化策略.

1. 使用 `Performance` 录制应用快照，查看调用情况

2. 查看`network`中网络请求情况，是否有资源因过大请求阻塞，导致后续资源无法加载，这种情况一般选择**分包**或**固定资源选择cdn分担**（多域名。浏览器设置的`http2.0`以下同域名仅允许同时最多6个tcp的限制）
可以通过 `React Developer Tools` 的 `Profiler` 的 `Flamegraph`（火焰图）或 `Ranked`（渲染时长排行榜） 查看各组件的渲染时长，根据对应的组件可以进行排查渲染问题，以下作答：

+ 通过检查代码中是否有重复触发的 `useEffect`

+ 检查是否有多次不同渲染周期中触发的`setState`导致的渲染（比如在一个事件中导致的state更新，导致依赖于该state的useEffect触发，而该effect中又有其他的setState，导致多个有依赖项的useEffect不同批次连环触发）

+ 检查是否在某个超大组件中需要渲染的元素过多，可使用子组件可考虑使用 `pureComponent`，或 `React.memo` ，或使用` useMemo`来根据依赖项更新子组件，或在父组件中将子组件需要的`props`通过使用`useMome`或`useCallback`缓存， 或在子组件中使用 `shouldComponentUpdate` 中校验是否需要更新来减少更新.

+  检查是否存在拖拽业务，这类业务一般会导致大量的`diff`存在，可以的话可以考虑不使用React的方式去实现，使用第三方非React的JS库去实现。

+ 同上情况，存在大量增删改查逻辑，会导致大量的的diff可以检查列表元素是否存在唯一的`key`，通过key可以让React复用Fiber从而避免重复创建` Fiber`节点与 `Dom `节点

+ 存在未清除掉的定时器或`dom`监听事件