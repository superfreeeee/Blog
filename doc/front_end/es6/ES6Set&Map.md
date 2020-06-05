# ES6: Set & Map

@[TOC](文章目錄)

<!-- TOC -->

- [ES6: Set & Map](#es6-set--map)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Set 集合](#set-集合)
        - [Set 創建](#set-創建)
        - [Set 的屬性和方法](#set-的屬性和方法)
        - [Set 遍歷操作](#set-遍歷操作)
        - [Set 應用](#set-應用)
            - [轉數組](#轉數組)
            - [數組去重](#數組去重)
            - [集合運算](#集合運算)
    - [WeakSet 弱集合](#weakset-弱集合)
        - [WeakSet 創建](#weakset-創建)
        - [WeakSet 實例方法](#weakset-實例方法)
        - [WeakSet 應用](#weakset-應用)
    - [Map 映射表](#map-映射表)
        - [前情提要：與一般對象 Object 的差異](#前情提要與一般對象-object-的差異)
        - [Map 創建](#map-創建)
        - [Map 的屬性和方法](#map-的屬性和方法)
        - [Map 遍歷操作](#map-遍歷操作)
        - [Map 類型轉換](#map-類型轉換)
            - [Map vs Array](#map-vs-array)
            - [Map vs Object](#map-vs-object)
            - [Map vs JSON](#map-vs-json)
    - [WeakMap 弱映射表](#weakmap-弱映射表)
        - [WeakMap 方法](#weakmap-方法)
        - [應用](#應用)
- [結語](#結語)

<!-- /TOC -->

## 簡介

ES6 提供了兩種新的數據結構，`Set集合`和 `Map映射表`，擴展了 JS 編程的靈活性。相信使用過其他語言的人對這兩個都不太陌生，我們就直接進入重點。

## 參考

<table>
    <tr>
        <td>ES6 set和map数据结构</td>
        <td><a href="http://caibaojian.com/es6/set-map.html">http://caibaojian.com/es6/set-map.html</a></td>
    </tr>
</table>

# 正文

## Set 集合

### Set 創建

集合就是保存不重複元素的數據結構，理所當然的，所謂的重複是使用嚴格相等運算符(`===`)比較的結果，上代碼

```js
// 使用數組創建
let s1 = new Set([1, 2, 3, 2, 1])
// s1 = Set { 1, 2, 3 }
```

```js
// 使用 add 添加
let s2 = new Set()
;[1, 2, 3, 2, 1].map((x) => s2.add(x))
// s2 = Set { 1, 2, 3 }
```

- 注意：Set 的比較方式還是與 `===` 有一定的差異，就是在 `NaN` 的比較上，如下

```js
NaN === NaN // false

let s = new Set([NaN, NaN]) // s = Set { NaN }
```

### Set 的屬性和方法

- `Set.prototype.constructor`：構造方法，默認為 `Set` 本身
- `Set.prototype.size`：Set 的大小
- `add(val)`：添加元素，返回集合本身
- `delete(val)`：刪除元素，返回刪除結果(false 表示元素不存在)
- `has(val)`：檢查元素是否存在，返回布林值
- `clear()`：清空集合

```js
let s = new Set([1, 2, 3, 2, 1])

s.size // 3

s.add(6) // s = Set { 1, 2, 3, 6 }

s.has(6) // true
s.has(8) // false

s.delete(6) // true
s.delete(8) // false

s.clear() // s = Set {}
```

### Set 遍歷操作

Set 內容的順序等同於插入的順序，隱式的保存了先後順序關係

- `keys()`：返回鍵的迭代器
- `values()`：返回值的迭代器
- `entries()`：返回鍵值對的迭代器
- `forEach(callback)`：透過回調函數訪問每個成員

```js
let s = new Set(['red', 'blue', 'green'])
for (let key of s.keys()) {
  console.log(key)
}
// red
// blue
// green

for (let val of s.values()) {
  console.log(val)
}
// red
// blue
// green

for (let [key, val] of s.entries()) {
  console.log(key, val)
}
// red red
// blue blue
// green green

s.forEach((key, value, s) => {
  console.log(key)
  console.log(value)
  console.log(s)
})
// red
// red
// Set { 'red', 'blue', 'green' }
// blue
// blue
// Set { 'red', 'blue', 'green' }
// green
// green
// Set { 'red', 'blue', 'green' }

Set.prototype.values === Set.prototype[Symbol.iterator] // true
Set.prototype.keys === Set.prototype[Symbol.iterator] // true
for (let value of s) {
  console.log(value)
}
// red
// blue
// green
```

- 說明 1：Set 的數據結構本質上就是一個鍵和值相同的鍵值對，所以 `keys()`、`values()` 方法的返回值沒有差異
- 說明 2：`forEach(callback)` 方法中的 `callback` 函數接受三個參數，依序為鍵、值和集合本身
- 說明 3：Set 的默認迭代器與 `keys`、`values` 返回的迭代器相同，所以遍歷 Set 時可以忽略 `values()` 方法，見最後一個示例

### Set 應用

#### 轉數組

將 Set 結構轉成數組類型

```js
let s = new Set([1, 2, 3, 2, 1])

// 1. 使用 Array.from
let a = Array.from(s)
// a = [ 1, 2, 3 ]

// 2. 使用展開運算符 ...
let a2 = [...s]
// a2 = [ 1, 2, 3 ]
```

#### 數組去重

可將數組轉換成 Set 再換回數組，即可達到去重的效果

```js
let array = [1, 2, 3, 2, 1]
// array = [ 1, 2, 3, 2, 1 ]
array = [...new Set(array)]
// array = [ 1, 2, 3 ]
```

#### 集合運算

可以透過 Set 和 Array 的方法結合，計算`並集(Union)`、`交集(Intersect)`、`差集(Difference)`

```js
let a = new Set([1, 2, 3])
let b = new Set([5, 4, 3])

// 並集
let union = new Set([...a, ...b])
// union = Set { 1, 2, 3, 5, 4 }

// 交集
let intersect = new Set([...a].filter((x) => b.has(x)))
// intersect = Set { 3 }

// 差集
let difference = new Set([...a].filter((x) => !b.has(x)))
// difference = Set { 1, 2 }
```

## WeakSet 弱集合

WeakSet 和 Set 類似，但是有兩個差別：

1. WeakSet 只能保存對象作為元素
2. 垃圾回收機制不考慮 WeakSet 中的引用，即 WeakSet 中引用的對象可能會在運行時消失，好處是不用擔心 WeakSet 所持有的引用造成的內存泄露問題

### WeakSet 創建

WeakSet 構造函數接受一個可遍歷的參數，保存內容必須是 object 對象，數組會被轉換成類數組對象

```js
let ws = new WeakSet([
  [3, 4],
  [1, 2]
])
let ws2 = new WeakSet([{ 0: 1 }, { 1: 2 }])
```

### WeakSet 實例方法

- `WeakSet.prototype.add(val)`：添加元素
- `WeakSet.prototype.delete(val)`：刪除元素
- `WeakSet.prototype.has(val)`：檢查元素存在

基本上都跟 Set 一樣，就省略代碼了

### WeakSet 應用

WeakSet 不能遍歷，因為不能保證遍歷的過程元素是否存在，所以基本上都要透過 has 確認後才能引用。簡單來說，WeakSet 就好比是一個紀錄，註冊到 WeakSet 上的對象會保留引用，而透過 has 方法就能檢驗元素是否存在。

## Map 映射表

### 前情提要：與一般對象 Object 的差異

與其他高級語言不同的是，JS 的 Object 本身就是一種鍵值對的集合，那為什麼還要設計 Map 結構呢？

JS 原本的 Object 對象，只能使用數字、字符串和 Symbol 三種類型作為鍵名，即便是透過對象擴展方法傳入其他類型，還是會被轉成字符串作為鍵名，如下

```js
let key = { name: 'John' }
let val = { age: 18 }
let obj = {
  [key]: val
}
// obj = { '[object Object]': { age: 18 } }
```

Map 則可以以任意類型的數據作為鍵名

```js
let key = { name: 'John' }
let val = { age: 18 }
let map = new Map()
map.set(key, value)
// map = Map { { name: 'John' } => { age: 18 } }
```

### Map 創建

Map 構造方法可以傳入一個數組，數組包含表示鍵值對的長度為二的數組作為元素；或是創建空的 Map 對象在透過 set 加入鍵值對

```js
let key = { name: 'key' }
let value = { name: 'value' }
let map = new Map([
  [1, 2],
  [key, value]
])
// map = Map { 1 => 2, { name: 'key' } => { name: 'value' } }

let map2 = new Map()
map2.set(1, 2)
map2.set(key, value)
// map2 = Map { 1 => 2, { name: 'key' } => { name: 'value' } }
```

### Map 的屬性和方法

- `Map.prototype.size`：鍵值對數量
- `Map.prototype.set(key, value)`：設置鍵值對（新增或覆蓋原有的鍵）
- `Map.prototype.get(key)`：透過鍵取值
- `Map.prototype.has(key)`：檢查鍵是否存在（注意！這邊使用嚴格等於運算符，對象作為鍵的時候必須引用相同）
- `Map.prototype.delete(key)`：刪除鍵值對
- `Map.prototype.clear()`：清空 Map 結構

```js
let key1 = { name: 'key1' }
let value1 = { name: 'value1' }
let key2 = [1, 2, 3]
let value2 = [1, 2, 3]

let map = new Map()

map.set(key1, value1).set(key2, value2).set(1, 2).set(true, false)
// map = Map {
//   { name: 'key1' } => { name: 'value1' },
//   [ 1, 2, 3 ] => [ 1, 2, 3 ],
//   1 => 2,
//   true => false
// }

map.get(key1) // { name: 'value1' }
map.get(true) // false

map.has(key1) // true
map.has({ name: 'key1' }) // false

map.delete(key2) // true
map.delete([1, 2, 3]) // false

map.clear() // map = Map {}
```

### Map 遍歷操作

- `keys()`：返回鍵的迭代器
- `values()`：返回值的迭代器
- `entries()`：返回鍵值對的迭代器
- `forEach(callback)`：透過回調函數訪問每個成員

與 Set 不同有兩點：

1. Map 的鍵值不相等，其餘表現幾乎與 Set 的遍歷操作差不多
2. Map 的默認迭代器與 `entries` 的方法相同而與 `keys`、`values` 方法不同

```js
Map.prototype.keys === Map.prototype[Symbol.iterator] // false
Map.prototype.values === Map.prototype[Symbol.iterator] // false
Map.prototype.entries === Map.prototype[Symbol.iterator] // true
```

### Map 類型轉換

#### Map vs Array

- Map 轉 Array

```js
let map = new Map([
  [{ name: 'key1' }, { name: 'value1' }],
  [true, false],
  [1, 2]
])
let array = [...map]
// array = [
//     [ { name: 'key1' }, { name: 'value1' } ],
//     [ true, false ],
//     [ 1, 2 ]
// ]
```

- Array 轉 Map

```js
let array = [
    [ { name: 'key1' }, { name: 'value1' } ],
    [ true, false ],
    [ 1, 2 ]
]
let map = new Map(array)
// Map {
//     { name: 'key1' } => { name: 'value1' },
//     true => false,
//     1 => 2
}
```

#### Map vs Object

- Map 轉 Object

前提：Map 的所有鍵都必須是字符串，以避免鍵重複

```js
function mapToObject(map) {
  let obj = {}
  for (let [key, value] of map) {
    obj[key] = value // 風險操作，key 為對象時轉為可能重複
  }
  return obj
}

let map = new Map([
  [{ name: 'key1' }, { name: 'value1' }],
  [true, false],
  [1, 2]
])
let obj = mapToObject(map)
// obj = { '1': 2, '[object Object]': { name: 'value1' }, true: false }
```

- Object 轉 Map

```js
function objectToMap(obj) {
  let map = new Map()
  for (let key in obj) {
    map.set(key, obj[key])
  }
  return map
}
let obj = {
  '1': 2,
  '[object Object]': { name: 'value1' },
  true: false
}
let map = objectToMap(obj)
// map = Map {
//   '1' => 2,
//   '[object Object]' => { name: 'value1' },
//   'true' => false
// }
```

#### Map vs JSON

- Map 轉 JSON

可以區分成兩種情況：

1. 鍵名都是字符串：轉換成對象 JSON
2. 鍵名存在非字符串：轉換成數組 JSON

```js
// case 1
let map = new Map([
  ['key1', 1],
  ['key2', { value: 2 }]
])
function mapToObject(map) {
  let obj = {}
  for (let [key, value] of map) {
    obj[key] = value // 風險操作，key 為對象時轉為可能重複
  }
  return obj
}
function mapToJSON(map) {
  return JSON.stringify(mapToObject(map))
}
let obj = mapToJSON(map)
// obj = {
//   key1: 1,
//   key2: {
//     value: 2
//   }
// }

// case 2
let map = new Map([
  [{ name: 'key1' }, 1],
  [{ value: 'key2' }, { value: 2 }]
])
function mapToArrJSON(map) {
  return JSON.stringify([...map])
}
let obj = mapToArrJSON(map)
// obj = [
//   [{ name: 'key1' }, 1],
//   [{ value: 'key2' }, { value: 2 }]
// ]
```

- JSON 轉 Map

也要區分成兩種情況：

1. 對象構成的 JSON
2. 數組構成的 JSON

```js
// case 1
function objectToMap(obj) {
  let map = new Map()
  for (let key in obj) {
    map.set(key, obj[key])
  }
  return map
}
function objJSONToMap(objJSON) {
  return objectToMap(JSON.parse(objJSON))
}
let objJSON = '{"key1":1,"key2":{"value":2}}'
let map = objJSONToMap(objJSON)
// map = Map { 'key1' => 1, 'key2' => { value: 2 } }

// case 2
function arrJSONToMap(arrJSON) {
  return new Map(JSON.parse(arrJSON))
}
let arrJSON = '[[{"name":"key1"},1],[{"value":"key2"},{"value":2}]]'
let map = arrJSONToMap(arrJSON)
// Map { { name: 'key1' } => 1, { value: 'key2' } => { value: 2 } }
```

## WeakMap 弱映射表

WeakMap、Map 之間的差異與 WeakSet、Set 之間雷同。WeakMap 只能用對象作為鍵名。

### WeakMap 方法

- `get(key)`：取值
- `set(key, value)`：設置鍵值對
- `has(key)`：檢查鍵是否存在
- `delete(key)`：刪除鍵值對

### 應用

可以用 WeakMap 保留 HTML 元素的事件監聽器，當元素消失時，監聽器就會自動被垃圾回收器回收。

# 結語

本篇介紹了 ES6 提供的兩種新的數據結構：Set、Map。Set 能實現去重，而 Map 提供了比一般對象更靈活的映射表。兩種新的數據結構的出現使程序員在算法選擇和實現上多了新的選項，也省去建立這兩種基本數據結構的成本。
