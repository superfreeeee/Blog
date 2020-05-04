# `Object.freeze()` vs `Object.seal()` vs `Object.preventExtensions()`

<!-- TOC -->

- [`Object.freeze()` vs `Object.seal()` vs `Object.preventExtensions()`](#objectfreeze-vs-objectseal-vs-objectpreventextensions)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [`Object.freeze(obj)`](#objectfreezeobj)
        - [語法](#語法)
        - [作用](#作用)
        - [Sample](#sample)
        - [`Object.isFrozen(obj)`](#objectisfrozenobj)
    - [`Object.seal(obj)`](#objectsealobj)
        - [語法](#語法-1)
        - [作用](#作用-1)
        - [Sample](#sample-1)
        - [`Object.isSealed(obj)`](#objectissealedobj)
    - [`Object.preventExtensions(obj)`](#objectpreventextensionsobj)
        - [語法](#語法-2)
        - [作用](#作用-2)
        - [Sample](#sample-2)
        - [`Object.isExtensible(obj)`](#objectisextensibleobj)
    - [Comparison 比較](#comparison-比較)
- [結語](#結語)

<!-- /TOC -->

## 簡介

ES5 除了提供 `Object.defineProperty` 方法使程序員能更精準的控制對象中鍵值對的表現，但有時候這也會帶來一些麻煩。可能一些對象你希望他維持著你設定好的樣式，這時候就要用到 ES5 提供的另外三個方法：`freeze`、`seal`、`preventExtensions`

## 參考

<table>
    <tr>
        <td>Object.freeze() vs Object.seal() vs Object. preventExtensions()</td>
        <td><a href="https://medium.com/@nlfernando11/object-freeze-vs-object-seal-vs-object-preventextensions-251ee99d0c47">https://medium.com/@nlfernando11/object-freeze-vs-object-seal-vs-object-preventextensions-251ee99d0c47</a></td>
    </tr>
</table>

# 正文

## `Object.freeze(obj)`

### 語法

```js
Object.freeze(obj)
```

### 作用

1. 禁止添加新屬性
2. 禁止移除舊屬性
3. 禁止修改舊屬性
4. 所有屬性的 `writable` 和 `configurable` 被改為 `false`，即對象中所有屬性不可修改也不可再配置
5. 禁止修改對象的 `prototype`(即 `__proto__` 屬性)

### Sample

```js
let obj = {
  id: 0,
  name: 'John'
}
```

```js
Object.freeze(obj)
// 禁止添加新屬性
obj.age = 18 // 無效，嚴格模式報錯

// 禁止移除舊屬性
delete obj.id // 無效，嚴格模式報錯

// 禁止修改舊屬性
obj.name = 'Sandy' // 無效，嚴格模式報錯

// 所有屬性的 `writable` 和 `configurable` 被改為 `false`，即對象中所有屬性不可修改也不可再配置
Object.getOwnPropertyDescriptor(obj, 'id')
// { value: 0, writable: false, enumerable: true, configurable: false }

// 禁止修改對象 `prototype`(即 `__proto__` 屬性)
obj.__proto__ = Function // 報錯
```

### `Object.isFrozen(obj)`

檢查對象是否被 freeze

```js
let obj = {
  id: 0,
  name: 'John'
}
Object.freeze(obj)
console.log(Object.isFrozen(obj))
// true
```

## `Object.seal(obj)`

### 語法

```js
Object.seal(obj)
```

### 作用

1. 禁止添加新屬性
2. 禁止移除舊屬性
3. 所有屬性的 `configurable` 被改為 `false`，即對象中所有屬性不可再被配置
4. 禁止修改舊屬性為 getter/setter

### Sample

```js
let obj = {
  id: 0,
  name: 'John'
}
```

```js
Object.seal(obj)
// 禁止添加新屬性
obj.age = 18 // 無效，嚴格模式報錯

// 禁止移除舊屬性
delete obj.id // 無效，嚴格模式報錯

// 所有屬性的 `configurable` 被改為 `false`，即對象中所有屬性不可再被配置
Object.getOwnPropertyDescriptor(obj, 'id')
// { value: 0, writable: true, enumerable: true, configurable: false }

// 禁止修改舊屬性為 getter/setter
Object.defineProperty(obj, 'name', {
  // 報錯
  get() {
    return this.name
  },
  set(newName) {
    this.name = newName
  }
})
```

### `Object.isSealed(obj)`

檢查對象是否被 seal

```js
let obj = {
  id: 0,
  name: 'John'
}
Object.seal(obj)
console.log(Object.isSealed(obj))
// true
```

## `Object.preventExtensions(obj)`

### 語法

```js
Object.preventExtensions(obj)
```

### 作用

1. 禁止添加新屬性

相較於前兩個方法，`preventExtensions` 是最弱的，僅僅只能防止添加新屬性

### Sample

```js
let obj = {
  id: 0,
  name: 'John'
}
```

```js
Object.preventExtensions(obj)
// 禁止添加新屬性
obj.age = 18 // 無效，嚴格模式報錯
```

### `Object.isExtensible(obj)`

檢查對象是否被 preventExtensions

```js
let obj = {
  id: 0,
  name: 'John'
}
Object.preventExtensions(obj)
console.log(Object.isExtensible(obj))
// true
```

## Comparison 比較

經過上述說明，我們可以發現三個方法都圍繞在一些相似的性質上

1. 禁止添加新屬性
2. 禁止移除舊屬性
3. 禁止修改舊屬性
4. `writable` 改為 `false`
5. `configurable` 改為 `false`
6. 禁止修改對象的 `prototype`(即 `__proto__` 屬性)
7. 禁止修改舊屬性為 getter/setter

以此我們可以繪製下列表格

| Feature                                         | Object.freeze() | Object.seal() | Object.preventExtensions() |
| ----------------------------------------------- | :-------------: | :-----------: | :------------------------: |
| 禁止添加新屬性                                  |        v        |       v       |             v              |
| 禁止移除舊屬性                                  |        v        |       v       |                            |
| 禁止修改舊屬性                                  |        v        |               |                            |
| `writable` 改為 `false`                         |        v        |               |                            |
| `configurable` 改為 `false`                     |        v        |       v       |                            |
| 禁止修改對象的 `prototype`(即 `__proto__` 屬性) |        v        |               |                            |
| 禁止修改舊屬性為 getter/setter                  |        v        |       v       |                            |

# 結語

ES5 提供了 `defineProperty` 透露出更多對象的屬性描述，也提供了 `freeze`、`seal`、`preventExtensions` 來限制對象的可修改性和擴展性，雖然平常的業務編程幾乎很少用到這些方法，但是許多響應式框架的背後都透過這些方法來擴展並限制對象，以此來確保框架的封裝和可靠性。
