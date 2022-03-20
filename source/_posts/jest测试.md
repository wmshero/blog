---
title: jest测试
date: 2022-01-20 11:52:39
tags: ['测试']
categories: '测试'
---


# 单元测试和集成测试
* 单元测试： 对软件的最小可测试单元进行检查和验证。前端所说的单元测试就是对一个模块进行测试，也就是说前端测试的时候，你测试的东西一定是一个模块。
* 集成测试：也叫组装测试或者联合测试。在单元测试的基础上，将所有模块按照涉及要求组装成子系统或系统，进行集成测试。

# 常用匹配符
```js
const dabaojian = require('./index.js')
const { test1 } = dabaojian

test('付款 300元', () => {
    expect((test1(300))).toBe('贵了')
})


test('测试严格相等', () => {
    const a = { number: '007' }
    expect(a).toEqual({ number: '007' })
})


test('测试严格相等', () => {
    const a = { number: '007' }
    expect(a).toEqual({ number: '007' })
})

test('toBeNull测试', () => {
    const a = null
    expect(a).toBeNull()
})

test('toBeUndefined测试', () => {
    const a = undefined
    expect(a).toBeUndefined()
})

// 只要定义过了，都可以匹配成功
test('toBeDefined测试', () => {
    const a = '你好'
    expect(a).toBeDefined()
})

//判断是否为真，null 和 undefined ,false,0无法通过，其他都是可以通过测试
test('toBeTruthy测试', () => {
    const a=1
    // const a=0
    // const a=true
    // const a={} 
    // const a=[]
    // const a='hi'
    expect(a).toBeTruthy()
})

test('toBeFalsy测试', () => {
    const a=0
    // const a=false
    // const a=null 
    // const a=undefined 
    expect(a).toBeFalsy()
})

// 大于
test('toBeGreaterThan',()=>{
    const count=10
    expect(count).toBeGreaterThan(9)
})

// 小于
test('toBeLessThan',()=>{
    const count=10
    expect(count).toBeLessThan(11)
})

// 大于或等于 （小于等于 toBeLessThanOrEqual）
test('toBeGreaterThanOrEqual',()=>{
    const count=11
    expect(count).toBeGreaterThanOrEqual(11)
})

// 消除js浮点精度错误的匹配器
test('toBeCloseTo',()=>{
    const num1=0.1
    const num2=0.2
    expect(num1+num2).toBeCloseTo(0.3)
})


// 字符串匹配
test('toMatch',()=>{
    const str='苹果，荔枝，栗子'
    expect(str).toMatch('苹果')
    // expect(str).toMatch(/荔枝/)
})



// 数组匹配
test('toContain',()=>{
    const arr=['苹果','荔枝','栗子']
    const data=new Set(arr)
    expect(data).toContain('苹果')
})


// 异常处理
test('toThrow',()=>{
  const throwNewErrorFunc=()=>{
      throw new Error('this is a new error')
  }
    expect(throwNewErrorFunc).toThrow()
    expect(throwNewErrorFunc).toThrow('this is a new error')
})

// 相反/取反
test('not toThrow',()=>{
    const throwNewErrorFunc=()=>{
        throw new Error('this is a new error')
    }
      expect(throwNewErrorFunc).not.toThrow('this is a new error')
  })
```

# 异步处理


```js

test('fetchData测试', (done) => {
    fetchData(data => {
        expect(data).toEqual({
            success: true
        })
        done()
    })
})


test('fetchData1测试', () => {
    expect.assertions(1) //断言，必须需要执行一次expect方法才可以通过测试
    return fetchData1.then(res => {
        expect(res.data).toEqual({
            success: true
        })
    }).catch(e=>{
        expect(e.toString().indexOf('404')>-1).toEqual(true)
    })
})


test('catch 测试', () => {
    expect.assertions(1)  // 断言，必须执行一次expect
    return fetchData1().catch((e)=>{
      expect(e.toString().indexOf('404')> -1).toBe(true)
    })
})

// async...await 测试
test('catch 测试', async() => {
    await expect(fetchData1()).resolves.toMatchObject({
        data:{
            success:true
        }
    })

    // 或者

    const res=await fetchData1()
    expect(res.data).toEqual({
        success:true
    })
})

```

# jest中的四个钩子函数

## beforeAll 
在所有测试用例之前进行执行

```js
beforeAll(()=>{
    console.log('beforeAll 在所有测试用例之前进行执行')
})
```
## afterAll 
在所有测试用例之后进行执行

```js
beforeAll(()=>{
    console.log('afterAll 在所有测试用例之后进行执行')
})
```

## beforeEach 
在每个测试用例前都会执行一次

```js
beforeEach(()=>{
    console.log('beforeEach 在每个测试用例前都会执行一次')
})
```


## afterEach 
在每个测试用例后都会执行一次

```js
afterEach(()=>{
    console.log('afterEach 在每个测试用例前都会执行一次')
})
```


# 测试用例分组
`describe()`分组


```js
describe('数字相关', () => {

    // 大于
    test('toBeGreaterThan', () => {
        const count = 10
        expect(count).toBeGreaterThan(9)
    })

    // 小于
    test('toBeLessThan', () => {
        const count = 10
        expect(count).toBeLessThan(11)
    })

    // 大于或等于 （小于等于 toBeLessThanOrEqual）
    test('toBeGreaterThanOrEqual', () => {
        const count = 11
        expect(count).toBeGreaterThanOrEqual(11)
    })

    // 消除js浮点精度错误的匹配器
    test('toBeCloseTo', () => {
        const num1 = 0.1
        const num2 = 0.2
        expect(num1 + num2).toBeCloseTo(0.3)
    })
})


describe('异步相关', () => {

    test('fetchData测试', (done) => {
        fetchData(data => {
            expect(data).toEqual({
                success: true
            })
            done()
        })
    })



    test('fetchData1测试', () => {
        expect.assertions(1) //断言，必须需要执行一次expect方法才可以通过测试
        return fetchData1.then(res => {
            expect(res.data).toEqual({
                success: true
            })
        }).catch(e => {
            expect(e.toString().indexOf('404') > -1).toEqual(true)
        })
    })


    test('catch 测试', () => {
        expect.assertions(1)  // 断言，必须执行一次expect
        return fetchData1().catch((e) => {
            expect(e.toString().indexOf('404') > -1).toBe(true)
        })
    })

    // async...await 测试
    test('catch 测试', async () => {
        await expect(fetchData1()).resolves.toMatchObject({
            data: {
                success: true
            }
        })

        // 或者

        const res = await fetchData1()
        expect(res.data).toEqual({
            success: true
        })
    })
})

```


# 钩子函数的作用域

jest中钩子函数的作用域有下面三个特色：
* 钩子函数在父级分组可作用域子集，类似继承
* 钩子函数同级分组作用域互不干扰，各起作用
* 先执行外部的钩子函数，再执行内部的钩子函数