---
title: React Hook
date: 2022-02-20 18:54:36
tags: [React,hook]
categories: 'React'
---
# 入门

[官方文档](https://zh-hans.reactjs.org/docs/hooks-intro.html)，可以用一些方式来[快速校验](https://codepen.io/gaearon/pen/oWWQNa?editors=0010)学习成果。

基础 Hooks 库：[ahooks](https://ahooks.js.org/zh-CN/)，来补充一些基础 JS 能力的 Hooks 实现，有兴趣的话，可以看一下其源码。


最佳实践

在使用 React hooks 需遵循一些最佳实践：

react-hooks/exhaustive-deps

这是 React hooks 的 ESLint 规则中最核心的一条，要求开发者穷尽 hook 的依赖，以减少因为遗漏了数据依赖关系而导致的 bug（这种 bug 通常 debug 成本很高）。

开发者如果选择在某处禁用该规则，应注释具体原因，否则该处代码将难以维护。

请扩展阅读以下官方 QA：

1. [在依赖列表中省略函数是否安全？](https://zh-hans.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)
2. [如果我的 effect 的依赖频繁变化，我该怎么办？]

useMemo

useMemo（及 useCallback）很容易被滥用，通常因为开发者会犯过早优化的错误。性能优化总是会有成本，但并不总是带来好处。不必要的 useMemo 的使用会导致以下问题：

1. useMemo 本身会增加代码执行成本。
2. useMemo 有连锁效应，如果你的一个 useMemo 依赖了另一个数组/ Object /函数...，你可能会被迫对该依赖也使用 useMemo/useCallback。最后你会使用一连串的 useMemo。
3. useMemo 一旦被使用之后，就很难被清理。开发者很难评估它当时是为了解决什么问题，深入分析的成本也很高，因此最终就不会有人选择动它。

所以我们推荐的策略是，默认不使用 useMemo/useCallback，直到你确实遇到以下问题时才考虑使用：

1. 有依赖在重复触发不必要的副作用（比如重复发送请求）。
2. 页面有明显的性能问题（可能由过多的重渲染或者量级巨大的循环计算造成）。
  ○ 这也是开发者过早优化时潜意识中所担心的问题，但现代计算机的性能对于网页来说是过剩的，React 底层也对重渲染做了大量优化，因此绝大部分情况下开发者不需要提前优化，应等到问题真的出现再做解决。
3. 重渲染导致了一些特殊组件的 UI 异常（比如图表每次重渲染都播放初始化动画）。

> 在调查组件渲染和性能问题时，可以使用 [React Profiler](https://zh-hans.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)。

另一种误用 useMemo 的场景是用空依赖的 useMemo 来保证一个非 primitive 类型常量的引用不变。这是一个错误的使用，因为常量无关输入，不需要被“记住”。应该直接定义在组件外：

```js
// Bad
function Foo() {
  const list = useMemo(() => [1, 2, 3], [])
  return <Bar prop={list} />
}

// Good
const list = [1, 2, 3]
function Foo() {
  return <Bar prop={list} />
}

```