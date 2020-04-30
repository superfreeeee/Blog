# Object.defineProperty

<!-- TOC -->

- [Object.defineProperty](#objectdefineproperty)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [語法](#語法)
        - [參數說明](#參數說明)
        - [查詢屬性描述](#查詢屬性描述)
    - [Property Descriptor 屬性描述](#property-descriptor-屬性描述)
        - [configurable 可配置性](#configurable-可配置性)
            - [Sample](#sample)
        - [enumerable 可枚舉性](#enumerable-可枚舉性)
            - [Sample](#sample-1)
        - [value 值](#value-值)
            - [Sample](#sample-2)
        - [writable 可修改性](#writable-可修改性)
            - [Sample](#sample-3)
        - [Accessor Properties 屬性訪問器(getter/setter)](#accessor-properties-屬性訪問器gettersetter)
        - [Sample](#sample-4)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Vue, React 等多數前端框架在做動態綁定的時候都用到了觀察者模式以及數據劫持方法，本篇就從 `Object.defineProperty` 方法的角度切入，悉數 JS 對象的屬性到底都隱含了哪些特徵，讓我們能對 JS 對象屬性的訪問控制有更深刻的了解。

## 參考

<table>
    <tr>
        <td>MDN web docs</td>
        <td><a>https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty</a></td>
    <tr>
    </tr>
        <td>簡書</td>
        <td><a>https://www.jianshu.com/p/f0d9a0ca98f4</a></td>
    </tr>
</table>

# 正文

## 語法

```
Object.defineProperty(obj, prop, descriptor)
```

### 參數說明

- obj：被定義的對象
- prop：要定義或修改的 key(鍵或稱為屬性)
- descriptor：屬性描述

### 查詢屬性描述

```
Object.getgetOwnPropertyDescriptor(obj, key)
```

可檢視對象中指定屬性的描述

## Property Descriptor 屬性描述

defineProperty 中第三個參數的各個屬性詳解

### configurable 可配置性

定義數據是否可被刪除(delete)或修改其他屬性，只可以透過 defineProperty 將 writable 從 true 改成 false。

**默認值為 true**

#### Sample

```js
let obj = { id: 3, name: 'John' }
Object.defineProperty(obj, 'name', {
  configurable: false
})
delete obj.name // 嚴格模式下報錯
console.log(obj) // obj: { id: 3, name: 'John' }, 屬性未被刪除
```

### enumerable 可枚舉性

決定對象的某個屬性是否可被枚舉，即影響到 for in, Object.keys, JSON.stringify。

- **默認值：**
  - **用戶自訂義的屬性為 true**
  - **原形繼承的屬性為 false**
  - **透過 defineProperty 新增的屬性為 false**

#### Sample

```js
function printKeysInThreeWays(obj) {
  for (let key in obj) {
    console.log(key)
  }
  console.log(Object.keys(obj))
  console.log(JSON.stringify(obj))
}

let obj = { id: 3, name: 'john' }
printKeysInThreeWays(obj)
// id
// name
// [ 'id', 'name' ]

Object.defineProperty(obj, 'name', {
  enumerable: false
})
printKeysInThreeWays(obj)
// id
// [ 'id' ]
// {"id":3}
```

### value 值

定義屬性的值，未設置時默認為 undefined

**默認值為 undefined**

#### Sample

```js
let obj = { id: 3, name: 'john' }
Object.defineProperty(obj, 'age', {
  value: 3
})
console.log(obj)
// { id: 3, name: 'john' }
console.log(Object.getOwnPropertyDescriptor(obj, 'age'))
// { value: 3, writable: false, enumerable: false, configurable: false }
Object.defineProperty(obj, 'name', {
  enumerable: true
})
console.log(obj)
// { id: 3, name: 'john', company: undefined }
```

### writable 可修改性

設定一個屬性的可修改性，簡單來說就是 set 方法的開關

**默認值為 true**

#### Sample

```js
let obj = { id: 3, name: 'john' }
obj.name = 'Sandy'
Object.defineProperty(obj, 'name', {
  writable: false
})
obj.name = 'Bob'
console.log(obj)
// { id: 3, name: 'Sandy' }
```

### Accessor Properties 屬性訪問器(getter/setter)

顯示定義屬性的訪問和設置函數(getter/setter)，不可與 value 屬性同時出現，綁定變量時類似閉包的表現。

**默認值 不存在，不可與 value 同時出現**

### Sample

```js
let _name = 'John'
let obj = { age: 18 }
Object.defineProperty(obj, 'id', {
  value: 0,
  enumerable: true
})
Object.defineProperty(obj, 'name', {
  enumerable: true,
  get: function () {
    return _name
  },
  set: function (name) {
    _name = name
  }
})
let val = obj.age
Object.defineProperty(obj, 'age', {
  get: function () {
    console.log(`get age`)
    return val
  },
  set: function (newVal) {
    console.log(`set age = ${newVal}`)
    val = newVal
  }
})
console.log(obj)
// { age: 100, id: 0, name: "John" }
console.log(obj.name)
// John
console.log(obj.age)
// get age
// 18
obj.age = 100
// set age = 100
```

# 結語

本篇簡單介紹 JS Object 的各個屬性描述符的作用，同時隱隱約約能看出數據劫持的端倪，後面會更進階的介紹透過一些 Object 的方法操作屬性描述符、Vue2 使用的數據劫持原理等。
