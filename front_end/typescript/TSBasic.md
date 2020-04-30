# TypeScript 基礎

<!-- TOC -->

- [TypeScript 基礎](#typescript-基礎)
    - [TypeScript 簡介](#typescript-簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Basic Type 基礎類型](#basic-type-基礎類型)
        - [Sample 代碼範例](#sample-代碼範例)
    - [Type Assertion 類型斷言](#type-assertion-類型斷言)
        - [Sample](#sample)
- [結語](#結語)

<!-- /TOC -->

## TypeScript 簡介

由於 JavaScript 的動態類型，使得系統開發侷限在一定規模之內。TypeScript 作為 JavaScript 的超集，透過開發環境引入靜態類型檢查，再透過 Babel 於運行時編譯回 JavaScript 代碼，做到類型控制同時又不影響現有 JS 運行環境。

## 參考

<table>
    <tr>
        <td>ts 官方文檔</td>
        <td><a href="https://www.typescriptlang.org/docs/home.html">https://www.typescriptlang.org/docs/home.html</a></td>
    </tr>
</table>

# 正文

## Basic Type 基礎類型

| Type            | Description              |
| --------------- | ------------------------ |
| Boolean         | 布林                     |
| Number          | 數字                     |
| String          | 字符串                   |
| Array           | 數組                     |
| Tuple           | 元組                     |
| Enum            | 枚舉                     |
| Any             | 任意類型                 |
| Void            | 空類型                   |
| Null, Undefined | null, undefined 自成一類 |
| Never           | 不可達                   |
| Object          | 對象                     |

### Sample 代碼範例

```ts
// Boolean
let b: boolean = true

// Number
let n: number = 10

// String
let s: string = '12345'
let s: string = `12345`

// Array
let a: number[] = [1, 2, 3, 4, 5]
let a: Array<number> = [1, 2, 3, 4, 5]

// Tuple
let t: [number, string] = [0, 'John']

// Enum
enum Color {
  Green,
  Red,
  Blue
} // Green = 0, 默認從 0 開始
let c: Color = Color.Green // >> c = 0
enum Color {
  Green = 1,
  Red = 2,
  Blue = 4
}
let c: Color = Color.Red // >> c = 2
let cs: string = Color[2] // >> cs = 'Red'

// Any
let v: any = 3
v = '123'

// Void：= null | undefined
function except(): void {}

// Null, Undefined
let v: null = null
let v: undefined = undefined

// Never
function error(msg: string): never {
  new Error(msg)
}

// Object
declare function create(o: object | null): void
create({ a: 1 })
create()
```

## Type Assertion 類型斷言

- 當你明確知道變量的類型，或需要強制聲明類型的時候

### Sample

```ts
// angle-bracket 尖括號
let str: any = '12345'
let len: number = (<string>str).length

// as-syntax as語法
let str: any = '12345'
let len: number = (str as string).length
```

# 結語

本篇介紹了 TS 的緣由，以及 TS 內部所有的基本類型。
