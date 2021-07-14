# React Hook: 高级 Hook API

@[TOC](文章目录)

<!-- TOC -->

- [React Hook: 高级 Hook API](#react-hook-高级-hook-api)
- [前言](#前言)
- [正文](#正文)
  - [1. useReducer](#1-usereducer)
  - [2. useMemo](#2-usememo)
  - [3. useCallback](#3-usecallback)
  - [4. useRef](#4-useref)
  - [5. useImperativeHandle](#5-useimperativehandle)
  - [6. useLayoutEffect](#6-uselayouteffect)
  - [7. useDebugValue](#7-usedebugvalue)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

继[前一篇 - React 升级: Hook API 基础](https://blog.csdn.net/weixin_44691608/article/details/117533917)，介绍了三个基本的 Hook：useState、useEffect、useContext

今天要来介绍更多跟多延伸的高级 Hook API

# 正文

## 1. useReducer

第一个有点像是 redux 的简易版，原来的 useState 是这样用的

```ts
const [state, setState] = useState(initState)
```

而 `useReducer` 则是类似 redux 的提交方式来维护状态，适合比较复杂的状态更新逻辑

- `/src/hooks/useTimer.tsx`

首先我们先定义好一个简单的 reducer

```ts
import { useReducer, useState } from 'react'

type TimerAction =
  | { type: 'INCREMENT' }
  | { type: 'RESET' }
  | { type: '' }

const timerReducer = (count: number, action: TimerAction) => {
  switch (action.type) {
    case 'INCREMENT':
      return count + 1
    case 'RESET':
      return 0
    default:
      return count
  }
}
```

接下来将与 reducer 相关的逻辑封装成一个自定义 Hook

```ts
export default function useTimer() {
  const [count, dispatch] = useReducer(timerReducer, 0)

  const increment = () => dispatch({ type: 'INCREMENT' })

  const reset = () => dispatch({ type: 'RESET' })

  return { count, increment, reset }
}
```

- `/src/tests/TestUseReducer.tsx`

最后组件内部则可以无感知的使用改状态

```ts
import React from 'react'
import useTimer from '../hooks/useTimer'

const TestUseReducer = () => {
  const { count, increment, reset } = useTimer()
  return (
    <div>
      <h2>useReducer</h2>
      <h3>count: {count}</h3>
      <div>
        <button onClick={increment}>increment</button>
        <button onClick={reset}>reset</button>
      </div>
    </div>
  )
}

export default TestUseReducer
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useReducer.gif)

## 2. useMemo

第二个比较像是一种优化手段，根据依赖数组值来决定需不需要重新计算，因此尽量不要在回调函数内部体现顺序强相关的逻辑，也不可以依赖该函数的调用时机

首先我们稍微封装以下输入控件要用的钩子

- `/src/hooks/useInput.tsx`

```ts
import { ChangeEvent, ChangeEventHandler, useState } from 'react'

export default function useInput(
  initValue: string = ''
): [string, ChangeEventHandler<HTMLInputElement>] {
  const [value, setValue] = useState(initValue)

  const handleChange = (e: ChangeEvent<HTMLInputElement>) =>
    setValue(e.target.value)

  return [value, handleChange]
}
```

- `/src/tests/TestUseMemo.tsx`

接下来为导出量 z 调用一个缓存钩子，并依赖于 x、y 值的变化而更新

```ts
import React, { ChangeEventHandler, useMemo } from 'react'
import useInput from '../hooks/useInput'

export interface RowProps {
  label: string
  value: string
  onChange?: ChangeEventHandler<HTMLInputElement>
  disabled?: boolean
}

export const Row = (props: RowProps) => {
  const { label, ...rest } = props
  return (
    <div>
      <label>
        {label} <input type="text" {...rest} />
      </label>
    </div>
  )
}

const TestUseMemo = () => {
  const [x, changeX] = useInput()
  const [y, changeY] = useInput()

  const z = useMemo(() => {
    console.log(`recalculate concatXY = ${x + y}`)
    return x + y
  }, [x, y])

  return (
    <div>
      <h2>useCallback</h2>
      <Row label={'x:'} value={x} onChange={changeX} />
      <Row label={'y:'} value={y} onChange={changeY} />
      <Row label={'z:'} value={z} disabled />
    </div>
  )
}

export default TestUseMemo
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useMemo.gif)

## 3. useCallback

第三个与 useMemo 有点像，不过不一样的是 useMemo 是缓存回调结果，而 useCallback 则是缓存回调函数本身，可能可以使函数进行重新绑定相关变量

- `/src/tests/TestUseCallback.tsx`

```ts
import React, { useCallback } from 'react'
import useInput from '../hooks/useInput'
import { Row } from './TestUseMemo'

const TestUseCallback = () => {
  const [x, changeX] = useInput()
  const [y, changeY] = useInput()

  const concatXY = useCallback(() => {
    console.log(`concatXY = ${x + y}`)
    return x + y
  }, [x, y])

  return (
    <div>
      <h2>useCallback</h2>
      <Row label={'x:'} value={x} onChange={changeX} />
      <Row label={'y:'} value={y} onChange={changeY} />
      <Row label={'z:'} value={concatXY()} disabled />
    </div>
  )
}

export default TestUseCallback
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useCallback.gif)

## 4. useRef

第四个其实在第一篇已经略微提过了，就是一个 Hook 版本的 React.createRef，用于创建一个 ref 的函数

- `/src/tests/TestUseCallback.tsx`

```ts
import React, { useCallback, useEffect, useRef } from 'react'

const TestUseRef = () => {
  const inputRef = useRef<HTMLInputElement>()

  const handleChange = useCallback(() => {
    console.log(`value = ${inputRef.current.value}`)
  }, [inputRef])

  return (
    <div>
      <h2>useRef</h2>
      <input type="text" ref={inputRef} onChange={handleChange} />
    </div>
  )
}

export default TestUseRef
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useRef.gif)

## 5. useImperativeHandle

接下来这一个也是与 ref 有关的，我们知道上层组件想要传递 ref 给子组件的时候可以使用 React.forwardRef 来传递给函数组件，也可以透过别名的方式来传递 ref，甚至可以通过 render props 或 children 等方式直接将 ref 标记到目标标签上

然而不论是哪一种似乎都对多 ref 的支持不友好，甚至我们没办法定制对于外部的 ref 表现，这时候我们就可以用上 `useImperativeHandle` 来为上级组件定制一个统一接口约定的组件

- `/src/tests/TestUseImperativeHandle.tsx`

首先 useImperativeHandle 的用法大致如下，透过与 React.forwardRef 配合，将传递下来的 ref 作为第一个参数，第二个参数则是一个回调函数，返回的是对外部可见的 InnerRefElement 类型，第三个参数一样是依赖列表

```ts
interface InnerRefElement {
  hi: () => void
  value: () => void
}

interface InnerProps {
  onChange: () => void
}

const Inner = React.forwardRef((props: InnerProps, ref) => {
  const inputRef = useRef<HTMLInputElement>()

  useImperativeHandle(
    ref,
    (): InnerRefElement => ({
      hi: () => {
        console.log('say Hi')
      },
      value: () => {
        console.log(`value = ${inputRef.current.value}`)
      },
    }),
    [inputRef]
  )

  return (
    <input type="text" ref={inputRef} onChange={props.onChange} />
  )
})
```

接下来对于外部组件来说，就可以根据定制化的接口来使用这个 ref

```ts
const TestUseImperativeHandle = () => {
  const ref = useRef<InnerRefElement>()

  useEffect(() => {
    if (ref.current) {
      ref.current.hi()
    }
  }, [ref])

  const onChange = useCallback(() => {
    ref.current.value()
  }, [ref])

  return (
    <div>
      <h2>useImperativeHandle</h2>
      <Inner ref={ref} onChange={onChange} />
    </div>
  )
}

export default TestUseImperativeHandle
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useImperativeHandle.gif)

## 6. useLayoutEffect

useLayoutEffect 与 useEffect 是几乎一摸一样的，不过差别在于 useLayoutEffect 的调用时机是处于 DOM 渲染之前，也就是说会阻塞渲染

所以除非特别需要直接修改 DOM 的操作，否则尽量使用 useEffect 来先渲染再修改

- `/src/tests/TestUseLayoutEffect.tsx`

```ts
import React, { useEffect, useLayoutEffect } from 'react'

const TestUseLayoutEffect = () => {
  useEffect(() => {
    console.log('useEffect')
  }, [])

  useLayoutEffect(() => {
    console.log('useLayoutEffect')
  }, [])

  return (
    <div>
      <h2>useLayoutEffect</h2>
    </div>
  )
}

export default TestUseLayoutEffect
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useLayoutEffect.png)

## 7. useDebugValue

useDebugValue 比较特别的是它其实是一个开发专属的 Hook 负责为开发时在开发者工具内提供 Hook 的特定标签

- `/src/hooks/useNothing.tsx`

首先我们写一个自定义 Hook 并使用 useDebugValue 打上标签

```ts
import { useDebugValue } from 'react'

export default function useNothing(bool: boolean) {
  useDebugValue(bool ? 'Online' : 'Offline')
}
```

- `/src/tests/TestUseDebugValue.tsx`

然后我们在组件内用一下，就可以在开发者工具中看到效果了

```ts
import React, { useState } from 'react'
import useNothing from '../hooks/useNothing'

const TestUseDebugValue = () => {
  const [state, setState] = useState(false)

  useNothing(state)

  return (
    <div>
      <h2>useDebugValue</h2>
      <button onClick={() => setState(!state)}>Toggle</button>
    </div>
  )
}

export default TestUseDebugValue
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_advanced_useDebugValue.gif)

# 结语

# 其他资源

## 参考连接

| Title                      | Link                                                                                                           |
| -------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Hook API 索引 - React 官方 | [https://react.docschina.org/docs/hooks-reference.html](https://react.docschina.org/docs/hooks-reference.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_hook_advanced](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_hook_advanced)
