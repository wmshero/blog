---
title: formily-QA
date: 2022-3-20 18:08:09
tags: ['formily']
categories: 'formily'
---

formily 是一个庞大且复杂的框架，且日常维护者也仅有几个，具有一定的学习成本，也有不少 bugs。因此在次设置一个文档用于记录问题和查询解决方案。


# QA

## 1. 为什么我改了 SchemaForm 的 initialValues 但不起作用？

`initialValues` 主要用于异步默认值场景，兼容同步默认值，只要在第 N 次渲染，某个字段还没被设置默认值，第 N+1 次渲染，就可以给其设置默认值


## 表单样式不符合？
Formily 所使用的是 antd 的样式。

可以通过传 `x-components-props: { style: {} }`，在 themed 里写覆盖样式等方法来解决。
如果样式修改很多，可考虑将原组件重新封装，但不建议重写。


如何用同一个 schema 渲染 Web 桌面端和 Web 移动端？
你需要参照 packages/src/components/Form/formily/fields/mobile.tsx 来创建一个支持双端渲染的 Field。

# 已知的 Bug

## 1. x-components-props 无法传递 moment object

建议直传 Date

## 2. 自定义校验规则不能直接使用 `x-rules: 'yourCustomRuleName'`

必须用 `x-rules: { yourCustomRuleName: true }`

## 使用 select 始终导出数组

单选型的情况下尽量使用 `Field { type: 'string', enums: [], }`，而不是 `select。`

## 选项型的 initialValues 会被清空

当 `initialValues` 的里存在有值的选项型，同时 `Schema` 尚未完全加载完毕（通常选项型对应的选项（Enum）是异步加载的，Schema 会经历一个初始化到逐渐加载完毕的过程）。那么在这个时候，Formily 会过滤掉无效的设置，导致 `FormState.initialValues` 的选项会被清空。

如果你需要完整的 `initialValues`，要么保存传入 `initialValues` 的接口数据，要么在 `Schema` 完全加载完毕后再渲染表单。

## x-component-props 无法传递将来可能会变化的值，如数组 index
