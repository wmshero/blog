---
title: styled component 使用规范
date: 2022-03-08 13:41:15
tags: [style,style component]
categories: 'style'
---
# 命名规范

## 定义组件

定义组件然后添加样式（通常再 export 出去）：以业务含义命名为基础，style 前加 `Raw` 后缀，style 后去除后缀。

```js
import styled from 'styled-components'

const HelpCenterRaw = () => {
  // define component
}

export const HelpCenter = styled(HelpCenterRaw)`
  // add styles
`
```

## 使用组件

import 后再次 style 一个组件，添加 Styled 后缀。

```js
import { HelpCenter } from './HelpCenter'
import styled from 'styled-components'

export const Header = () => ({
  <HelpCenterStyled />
})

const HelpCenterStyled = styled(HelpCenter)`
  // some styles
`
```

## Html Tag 或通用基础组件

有时我们 style 的不是自定义业务组件，而是 html tag 或通用基础组件，那就不存在重新命名的问题，以业务含义命名即可。

```js
import styled from 'styled-components'

export const Header = () => ({
  <Logo />
  <UserMenu />
})

const Logo = styled.img`
  // some styles
`
const UserMenu = styled(Menu)`
  // some style
`
```

# 其他规范

## 将样式代码放至文件末尾

基于业务逻辑比样式重要的原则，我们一般把样式代码放到文件的最后。上文代码即为示例。

覆盖 Ant Design 组件样式

当你尝试用 styled-components 去 wrap 一个 Ant Design 组件以覆盖原样式时，可能因为样式权重不够而失败。添加 !important 不是一个好的实践，可以按 styled-components 官方推荐的方法添加多个 & 来提高权重：

```js
const DividerStyled = styled(Divider)`
  &&& {
    margin: 0 12px 0 4px;
  }
`
```

## 在 React Native 中使用

用法上没有不同，但 import 方式稍有不同：

```js
import styled from 'styled-components/native' // instead of 'styled-components'
```

styled-components 内部使用了 css-to-react-native 来将 CSS 文本转化为 React Native StyleSheet Object，详见：https://github.com/styled-components/css-to-react-native

## 用 props 动态计算样式

```js
// Good

<Foo status={Status.Success} />

const Foo = styled.span<{ status?: Status }>`
  font-weight: bold;
  background-color: ${({ status }) => {
    switch (status) {
      case StatusEnum.Success:
        return Color.Green
      case StatusEnum.Warning:
        return Color.Yellow
      default:
        return Color.Gray
    }
  }};
`

// Bad

const Foo = styled.span.attrs({ status }: { status?: Status }) => ({
  status,
}))`
  ...
`

/**
 * .attrs() 的主要用途是给 styled 的目标元素/组件透传一些属性（attributes），
 * 比如 styled.input.attrs(props => ({ type: 'password' }))` ... `，
 * 这个会设置 input 的 type 属性为 password，
 * 详见：https://styled-components.com/docs/basics#attaching-additional-props
 */


```

# FAQ

## 如何获得 styled-components 样式代码的语法高亮？

VSCode 可安装 jpoissonnier.vscode-styled-components 插件。
WebStorm 可安装 Styled Components & Styled JSX 插件。

## 在 React Native 中使用 StyleSheet 时，stylelint 报 selector-type-no-unknown 错误

请统一使用 styled-components 来定义样式。

## 在 React Native 中使用 styled-components 时，stylelint 报 property-no-unknown 错误

将该 React Native 特有的样式属性添加到 stylelint.config.js 的 rules.property-no-unknown.ignoreProperties 中。

## 在 stylelint 报错的 line number 的对应行上找不到错误

stylelint 报错的 line number 是一个样式 block 的第一行，错误需要在整个 block 内寻找。

Warning: Received "true" for a non-boolean attribute
是因为传递的 true/false 不是符合标准的 DOM attribute，以下链接有更详细的说明
https://styled-components.com/docs/faqs#why-am-i-getting-html-attribute-warnings
https://github.com/styled-components/styled-components/issues/1198

`0` is not a valid style property

```js
const HeaderColumn = styled(View)<{ width?: number; }>`
  padding: 14px 0;
  align-items: center;

	// 错误写法，width 的值会被 StyleSheet 当作 style key
  ${props =>
    props.width &&
    css`
      width: ${props.width}px;
    `}

  // 正确写法，改用三元
	${props =>
    props.width ?
    css`
      width: ${props.width}px;
    ` : ''}
`
```