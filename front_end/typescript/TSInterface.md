# Interface 接口

<!-- TOC -->

- [Interface 接口](#interface-接口)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Basic 原始定義](#basic-原始定義)
    - [Optional Properties 可選屬性](#optional-properties-可選屬性)
    - [Readonly Properties 只讀屬性](#readonly-properties-只讀屬性)
        - [readonly vs const](#readonly-vs-const)
    - [Function Type 函數類型](#function-type-函數類型)
    - [Indexable Type 可索引類型](#indexable-type-可索引類型)
    - [Class Type 類](#class-type-類)
        - [公共部分和靜態部分區分](#公共部分和靜態部分區分)
    - [Extendsion 繼承](#extendsion-繼承)
    - [Hybird 混合](#hybird-混合)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Interface 是 TS 中非常重要的特性，相當於定義對象的每個詳細屬性已經一些訪問控制。

## 參考

<table>
    <tr>
        <td>ts 官方文檔</td>
        <td><a href="https://www.typescriptlang.org/docs/home.html">https://www.typescriptlang.org/docs/home.html</a></td>
    </tr>
</table>

# 正文

## Basic 原始定義

TS 會檢查 Interface 中定義的屬性是否存在，同時通常不允許出現未定義屬性。

```ts
interface User {
  id: number
  name: string
}

function introduction(user: User) {
  console.log(`user: {id = ${user.id}, name = ${user.name}}`)
}
let user: User = { id: 0, name: 'John' }
introduction(user)
```

## Optional Properties 可選屬性

可選的屬性，可存在可不存在。

```ts
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  const color: string = config.color ? config.color : 'black'
  const area: number = (config.width ? config.width : 100) * 20
  return { color, area }
}

const square = createSquare({ width: 100 })
// square = {color: "black", area: 2000}
```

## Readonly Properties 只讀屬性

只能讀，不能寫的屬性。

```ts
interface Point {
  readonly x: number
  readonly y: number
}

const point: Point = { x: 10, y: 20 }
point.x = 10 // error, x is read-only property
```

### readonly vs const

const 作為常量類型，readonly 作為屬性的特性。

## Function Type 函數類型

可透過 Interface 定義函數類型。

```ts
interface RegisterFunc {
  (id: number, name: string): { id: number; name: string }
}

const register: RegisterFunc = (id, name) => {
  return { id, name }
}
```

## Indexable Type 可索引類型

簡單來說就是限定索引方式的參數類型。

```ts
interface IndexByNumber {
  [index: number]: any
}

let io: IndexByNumber = ['1', '2', '3']
console.log(io[0])
console.log(io['0']) // error: '0' is not number type
```

## Class Type 類

類似 Java 裡面的接口定義，只檢查 Class 的公有部分，不檢查靜態部分（如構造函數），使用 implements 關鍵字。另外可將構造方法抽象成另一個函數並單獨定義接口。

```ts
interface Runnable {
  speed: number
  run(): void
}

class Man implements Runnable {
  speed: number
  constructor() {
    this.speed = 123
  }
  run() {
    console.log(`run with speed ${this.speed}`)
  }
}
```

### 公共部分和靜態部分區分

若想要定義構造函數類型，需要區分成構造方法接口和類接口。

```ts
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface
}

interface ClockInterface {
  tick(): void
}

function createClock(
  ctor: ClockConstructor,
  hour: number,
  minute: number
): ClockInterface {
  return new ctor(hour, minute)
}

class Clock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log('dang dang')
  }
}

let clock: ClockInterface = createClock(Clock, 12, 0)
```

## Extendsion 繼承

透過繼承的特性製造接口的階級制度。

```ts
interface Shape {
  color: string
}

interface Square extends Shape {
  sideLength: number
}

interface Rectangle extends Shape {
  width: number
  height: number
}

let square: Square = <Square>{}
square.color = 'green'
square.sideLength = 100
let rect: Rectangle = <Rectangle>{}
rect.width = rect.height = 50
```

## Hybird 混合

可定義一個同時作為函數和對象的類型。

```ts
interface Counter {
  (start: number): string
  interval: number
  reset(): void
}

function getCounter(): Counter {
  const counter = <Counter>function (start) {}
  counter.interval = 100
  counter.reset = function () {}
  return counter
}

let counter: Counter = getCounter()
counter(10)
counter.interval = 50
counter.reset()
```

# 結語

Interface 接口提供強大而靈活的類型定義方法，使得 TS 對於類型檢查的掌控更上一層樓。
