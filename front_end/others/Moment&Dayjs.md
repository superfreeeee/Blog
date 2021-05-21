# Moment.js & Day.js 时间日期类库

@[TOC](文章目录)

<!-- TOC -->

- [Moment.js & Day.js 时间日期类库](#momentjs--dayjs-时间日期类库)
- [前言](#前言)
- [正文](#正文)
  - [0. 环境配置](#0-环境配置)
    - [0.1 安装依赖](#01-安装依赖)
    - [0.2 辅助工具函数](#02-辅助工具函数)
  - [1. Moment.js](#1-momentjs)
    - [1.1 创建时间日期对象](#11-创建时间日期对象)
    - [1.2 获取/设置属性](#12-获取设置属性)
    - [1.3 日期操作(加、减、对齐、选择)](#13-日期操作加减对齐选择)
    - [1.4 查询(判断、比较)](#14-查询判断比较)
    - [1.5 格式化输出](#15-格式化输出)
  - [2. Day.js](#2-dayjs)
    - [2.1 创建时间日期对象](#21-创建时间日期对象)
      - [2.1.1 差异](#211-差异)
    - [2.2 获取/设置属性](#22-获取设置属性)
      - [2.2.1 差异](#221-差异)
    - [2.3 日期操作(加、减、对齐、选择)](#23-日期操作加减对齐选择)
      - [2.3.1 差异](#231-差异)
    - [2.4 查询(判断、比较)](#24-查询判断比较)
      - [2.4.1 差异](#241-差异)
    - [2.5 格式化输出](#25-格式化输出)
      - [2.5.1 差异](#251-差异)
  - [3. Day.js 相较于 Moment.js 的好处](#3-dayjs-相较于-momentjs-的好处)
    - [3.1 Moment.js 的问题](#31-momentjs-的问题)
    - [3.2 Day.js 的好处](#32-dayjs-的好处)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

在项目开发的时候，日期时间处理是一个非常常见的需求。然而问题在于 JS 原生的 Date 对象操作其实不太方便，因此延伸出了许多三方库用于封装 Date 对象，以及推动新的日期标准 `Intl` 的制定和实现。

本篇挑出三方库中大名鼎鼎的 moment.js 和 day.js 做介绍

# 正文

## 0. 环境配置

本篇实例运行在 node 环境，先来配置一下环境和依赖

### 0.1 安装依赖

```bash
$ yarn init -y  # 初始化项目
$ yarn add moment dayjs  # 安装核心依赖
```

以及运行测试命令

- `package.json`

```json
{
  "name": "moment-dayjs",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "dayjs": "node src/dayjs/index.js",
    "moment": "node src/moment/index.js"
  },
  "dependencies": {
    "dayjs": "^1.10.4",
    "moment": "^2.29.1"
  }
}
```

### 0.2 辅助工具函数

同时我们先定义一下后面会用到的工具函数，避免误解

- `src/utils/group.js`

```js
const emptyFn = () => {}

const log = console.log

function group(name, cb = emptyFn) {
  console.group(name)
  cb()
  console.groupEnd()
}

module.exports = {
  log,
  group,
}
```

`log` 为 `console.log` 的别名，`group` 方法对 `console.group` 稍微封装一下

## 1. Moment.js

第一个我们首先来介绍 Moment.js 提供的日期时间操作能力，主要分成以下几个大类介绍一些比较常见的功能需求和用法

- 创建时间日期对象
- 获取/设置属性
- 日期操作(加、减、对齐、选择)
- 查询(判断、比较)
- 格式化输出

### 1.1 创建时间日期对象

一开始当然是先创建 moment.js 定义的新的时间对象，直接使用 `moment` 方法构造

- `src/moment/1_parse.js`

```js
const moment = require('moment')
const { log, group } = require('../utils/group')

group('moment.js parse', () => {
  group('moment', () => {
    log(`moment():                            ${moment().format()}`)
    log(`moment('2021-11-01'):                ${moment('2021-11-01').format()}`)
    log(`moment('2021-11-01 23:59:59'):       ${moment('2021-11-01 23:59:59').format()}`)
    log(`moment('20211101T235959'):           ${moment('20211101T235959').format()}`)
    log(`moment('01-11-2021', 'DD-MM-YYYY'):  ${moment('01-11-2021', 'DD-MM-YYYY').format()}`)
    log(`moment([2021, 10, 1, 23, 59, 59]):   ${moment([2021, 10, 1, 23, 59, 59]).format()}`)
  })
  
// ...
```

传入的内容还是比较自由，主要以字符串描述为主，可用函数标签如下

```js
moment()        // 当前时间
moment(string)  // UTC 或 ISO 格式的时间描述字符串
moment(string, string)  // 第二个参数作为自定义字符串模版
moment(number)  // 传入时间戳
moment(array)   // 按序传入时间描述
```

看输出吧

```
moment.js parse
  moment
    moment():                            2021-05-21T14:37:10+08:00
    moment('2021-11-01'):                2021-11-01T00:00:00+08:00
    moment('2021-11-01 23:59:59'):       2021-11-01T23:59:59+08:00
    moment('20211101T235959'):           2021-11-01T23:59:59+08:00
    moment('01-11-2021', 'DD-MM-YYYY'):  2021-11-01T00:00:00+08:00
    moment([2021, 10, 1, 23, 59, 59]):   2021-11-01T23:59:59+08:00
```

还是比较简单直接的，另外由于 moment 对象的操作常常会直接改变对象实例，所以我们还需要另一个 `clone` 方法来拷贝对象进行复用

```js
// ...

  group('clone', () => {
    const m1 = moment()
    const m2 = moment(m1)
    const m3 = m2.clone()
    log(`m1:  ${m1.toString()}`)
    log(`m2:  ${m2.toString()}`)
    log(`m3:  ${m3.toString()}`)
    log(`m1 === m2:                        ${m1 === m2}`)
    log(`m1 === m3:                        ${m1 === m3}`)
    log(`m2 === m3:                        ${m2 === m3}`)
    log(`m1.toString() === m2.toString():  ${m1.toString() === m2.toString()}`)
    log(`m1.toString() === m3.toString():  ${m1.toString() === m3.toString()}`)
    log(`m2.toString() === m3.toString():  ${m2.toString() === m3.toString()}`)
  })
})
```

我们可以直接调用 `moment(instance)` 或是 `moment.clone()` 来创建实例副本

### 1.2 获取/设置属性

第二个模块则是介绍对于时间日期属性的获取和直接设置

首先我们给出几个函数标签

```js
// moment# 表示 moment.prototype 的原型方法
moment#[unit]()   // 获取指定单位(unit) 的值
moment#set(unit)  // 获取指定单位(unit) 的值
moment#[unit](val)     // 设置指定单位(unit) 的值
moment#set(unit, val)  // 设置指定单位(unit) 的值
```

我们可以透过直接调用指定单位来获取/设置属性值，或是调用 `get/set` 方法并传入目标属性字符串来进行调用

- `src/moment/2_getter_setter.js`

```js
const moment = require('moment')
const { log, group } = require('../utils/group')

group('moment.js getter/setter', () => {
  const origin = moment('2021-11-01 23:59:59.699')

  let time = origin.clone()

  group('second', () => {
    log(`time:              ${time}`)
    log(`time.second():     ${time.second()}`)
    log(`time.second(996):  ${time.second(30)}`)
    log(`time.second():     ${time.second()}`)
  })

  group('hour', () => {
    log(`time:          ${time}`)
    log(`time.hour():   ${time.hour()}`)
    log(`time.hour(1):  ${time.hour(1)}`)
    log(`time.hour():   ${time.hour()}`)
  })

  time = origin.clone()

  group('get/set', () => {
    log(`current time:             ${time}`)
    log(`time.get('year'):         ${time.get('year')}`)
    log(`time.get('month'):        ${time.get('month')}`)
    log(`time.get('date'):         ${time.get('date')}`)
    log(`time.get('day'):          ${time.get('day')}`)
    log(`time.get('hour'):         ${time.get('hour')}`)
    log(`time.get('minute'):       ${time.get('minute')}`)
    log(`time.get('second'):       ${time.get('second')}`)
    log(`time.get('millisecond'):  ${time.get('millisecond')}`)

    log(`time.set('year', 2022):      ${time.set('year', 2022)}`)
    log(`time.set('month', 0):        ${time.set('month', 0)}`)
    log(`time.set('date', 13):        ${time.set('date', 13)}`)
    log(`time.set('day', 1):          ${time.set('day', 1)}`)
    log(`time.set('hour', 1):         ${time.set('hour', 1)}`)
    log(`time.set('minute', 3):       ${time.set('minute', 3)}`)
    log(`time.set('second', 5):       ${time.set('second', 5)}`)
    log(`time.set('millisecond', 7):  ${time.set('millisecond', 7)}`)
  })
})
```

- 输出

```
moment.js getter/setter
  second
    time:              Mon Nov 01 2021 23:59:59 GMT+0800
    time.second():     59
    time.second(996):  Mon Nov 01 2021 23:59:30 GMT+0800
    time.second():     30
  hour
    time:          Mon Nov 01 2021 23:59:30 GMT+0800
    time.hour():   23
    time.hour(1):  Mon Nov 01 2021 01:59:30 GMT+0800
    time.hour():   1
  get/set
    current time:             Mon Nov 01 2021 23:59:59 GMT+0800
    time.get('year'):         2021
    time.get('month'):        10
    time.get('date'):         1
    time.get('day'):          1
    time.get('hour'):         23
    time.get('minute'):       59
    time.get('second'):       59
    time.get('millisecond'):  699
    time.set('year', 2022):      Tue Nov 01 2022 23:59:59 GMT+0800
    time.set('month', 0):        Sat Jan 01 2022 23:59:59 GMT+0800
    time.set('date', 13):        Thu Jan 13 2022 23:59:59 GMT+0800
    time.set('day', 1):          Mon Jan 10 2022 23:59:59 GMT+0800
    time.set('hour', 1):         Mon Jan 10 2022 01:59:59 GMT+0800
    time.set('minute', 3):       Mon Jan 10 2022 01:03:59 GMT+0800
    time.set('second', 5):       Mon Jan 10 2022 01:03:05 GMT+0800
    time.set('millisecond', 7):  Mon Jan 10 2022 01:03:05 GMT+0800
```

### 1.3 日期操作(加、减、对齐、选择)

第三个场景我们可能会需要对当前的时间对象进行加法、减法的操作，或是向某个时间单位的头尾进行对齐，以及多个时间单位选择最早/最晚的时间，可用的函数标签如下

```js
moment#add(val, unit)       // 在 unit 单位加上 val 值
moment#subtract(val, unit)  // 在 unit 单位减去 val 值
moment#startOf(unit)  // 向下取整到最近的 unit 单位
moment#endOf(unit)    // 向上取整到最近的 unit 单位
moment.max(...values)  // 取参数中最大(最晚)的时间
moment.min(...values)  // 取参数中最小(最早)的时间
```

- `src/moment/3_operate.js`

```js
const moment = require('moment')
const { log, group } = require('../utils/group')

group('moment.js operate', () => {
  const origin = moment('2019-11-01 23:59:59.699')

  let time = origin.clone()
  group('add', () => {
    log(`current time:            ${time}`)
    log(`time.add(365, 'days'):   ${time.add(365, 'days')}`)
    log(`reset time:              ${time.set('year', 2019)}`)
    log(`time.add(1, 'years'):    ${time.add(1, 'years')}`)
    log(`time.add(8, 'd'):        ${time.add(8, 'd')}`)
    log(`time.add({ days: 7 }):   ${time.add({ days: 7 })}`)
  })

  time = origin.clone().year(2020)
  group('subtract', () => {
    log(`current time:                ${time}`)
    log(`time.subtract(365, 'days'):  ${time.subtract(365, 'days')}`)
    log(`reset time:                  ${time.year(2020).date(1)}`)
    log(`time.subtract(1, 'years'):   ${time.subtract(1, 'years')}`)
  })

  group('startOf/endOf', () => {
    time = moment('2019-01-01 01:30:30.333')
    log(`time:                    ${time}`)
    log(`time.startOf('minute'):  ${time.clone().startOf('minute')}`)
    log(`time.startOf('hour'):    ${time.clone().startOf('hour')}`)

    log(`time:                  ${time}`)
    log(`time.endOf('minute'):  ${time.clone().endOf('minute')}`)
    log(`time.endOf('hour'):    ${time.clone().endOf('hour')}`)
  })

  group('max/min', () => {
    const laterOne = moment('2020-01-01 12')
    const earlyOne = moment('2020-01-01 05')
    log(`earlyOne:                                     ${earlyOne}`)
    log(`laterOne:                                     ${laterOne}`)

    log(`moment.max(earlyOne, laterOne):               ${moment.max(earlyOne, laterOne)}`)
    log(`moment.max(earlyOne, laterOne) === laterOne:  ${moment.max(earlyOne, laterOne) === laterOne}`)

    log(`moment.min(earlyOne, laterOne):               ${moment.min(earlyOne, laterOne)}`)
    log(`moment.min(earlyOne, laterOne) === earlyOne:  ${moment.min(earlyOne, laterOne) === earlyOne}`)
  })
})
```

- 输出

```js
moment.js operate
  add
    current time:            Fri Nov 01 2019 23:59:59 GMT+0800
    time.add(365, 'days'):   Sat Oct 31 2020 23:59:59 GMT+0800
    reset time:              Thu Oct 31 2019 23:59:59 GMT+0800
    time.add(1, 'years'):    Sat Oct 31 2020 23:59:59 GMT+0800
    time.add(8, 'd'):        Sun Nov 08 2020 23:59:59 GMT+0800
    time.add({ days: 7 }):   Sun Nov 15 2020 23:59:59 GMT+0800
  subtract
    current time:                Sun Nov 01 2020 23:59:59 GMT+0800
    time.subtract(365, 'days'):  Sat Nov 02 2019 23:59:59 GMT+0800
    reset time:                  Sun Nov 01 2020 23:59:59 GMT+0800
    time.subtract(1, 'years'):   Fri Nov 01 2019 23:59:59 GMT+0800
  startOf/endOf
    time:                    Tue Jan 01 2019 01:30:30 GMT+0800
    time.startOf('minute'):  Tue Jan 01 2019 01:30:00 GMT+0800
    time.startOf('hour'):    Tue Jan 01 2019 01:00:00 GMT+0800
    time:                  Tue Jan 01 2019 01:30:30 GMT+0800
    time.endOf('minute'):  Tue Jan 01 2019 01:30:59 GMT+0800
    time.endOf('hour'):    Tue Jan 01 2019 01:59:59 GMT+0800
  max/min
    earlyOne:                                     Wed Jan 01 2020 05:00:00 GMT+0800
    laterOne:                                     Wed Jan 01 2020 12:00:00 GMT+0800
    moment.max(earlyOne, laterOne):               Wed Jan 01 2020 12:00:00 GMT+0800
    moment.max(earlyOne, laterOne) === laterOne:  true
    moment.min(earlyOne, laterOne):               Wed Jan 01 2020 05:00:00 GMT+0800
    moment.min(earlyOne, laterOne) === earlyOne:  true
```

### 1.4 查询(判断、比较)

第四个场景是进行时间的判断、比较，给出函数标签

```js
moment#isBefore(other)  // 判断当前实例时间是否早于 other
moment#isSame(other)    // 判断当前实例时间是否等于 other
moment#isAfter(other)   // 判断当前实例时间是否晚于 other
moment#isBetween(from, to)   // 判断当前实例时间是否介于 from 和 to 之间
moment#isLeapYear()     // 判断是否为闰年
```

- `src/moment/4_query.js`

```js
const moment = require('moment')
const { log, group } = require('../utils/group')

group('moment.js query', () => {
  const earlyOne = moment('2020-01-01 05')
  const laterOne = moment('2020-01-01 12')
  const middleOne = moment('2020-01-01 08')

  log(`earlyOne:   ${earlyOne}`)
  log(`laterOne:   ${laterOne}`)
  log(`middleOne:  ${middleOne}`)

  group('isBefore', () => {
    log(`earlyOne.isBefore(laterOne):  ${earlyOne.isBefore(laterOne)}`)
    log(`laterOne.isBefore(earlyOne):  ${laterOne.isBefore(earlyOne)}`)
    log(`earlyOne.isBefore(earlyOne):  ${earlyOne.isBefore(earlyOne)}`)
  })

  group('isSame', () => {
    log(`earlyOne.isSame(laterOne):  ${earlyOne.isSame(laterOne)}`)
    log(`laterOne.isSame(earlyOne):  ${laterOne.isSame(earlyOne)}`)
    log(`earlyOne.isSame(earlyOne):  ${earlyOne.isSame(earlyOne)}`)
  })

  group('isAfter', () => {
    log(`earlyOne.isAfter(laterOne):  ${earlyOne.isAfter(laterOne)}`)
    log(`laterOne.isAfter(earlyOne):  ${laterOne.isAfter(earlyOne)}`)
    log(`earlyOne.isAfter(earlyOne):  ${earlyOne.isAfter(earlyOne)}`)
  })

  group('isBetween', () => {
    log(`middleOne.isBetween(earlyOne, laterOne):  ${middleOne.isBetween(earlyOne, laterOne)}`)
    log(`middleOne.isBetween(laterOne, earlyOne):  ${middleOne.isBetween(laterOne, earlyOne)}`)
  })

  group('isLeapYear', () => {
    log(`moment('1600').isLeapYear():  ${moment('1600').isLeapYear()}`)
    log(`moment('1800').isLeapYear():  ${moment('1800').isLeapYear()}`)
    log(`moment('2000').isLeapYear():  ${moment('2000').isLeapYear()}`)
    log(`moment('2020').isLeapYear():  ${moment('2020').isLeapYear()}`)
  })
})
```

- 输出

```
moment.js query
  earlyOne:   Wed Jan 01 2020 05:00:00 GMT+0800
  laterOne:   Wed Jan 01 2020 12:00:00 GMT+0800
  middleOne:  Wed Jan 01 2020 08:00:00 GMT+0800
  isBefore
    earlyOne.isBefore(laterOne):  true
    laterOne.isBefore(earlyOne):  false
    earlyOne.isBefore(earlyOne):  false
  isSame
    earlyOne.isSame(laterOne):  false
    laterOne.isSame(earlyOne):  false
    earlyOne.isSame(earlyOne):  true
  isAfter
    earlyOne.isAfter(laterOne):  false
    laterOne.isAfter(earlyOne):  true
    earlyOne.isAfter(earlyOne):  false
  isBetween
    middleOne.isBetween(earlyOne, laterOne):  true
    middleOne.isBetween(laterOne, earlyOne):  false
  isLeapYear
    moment('1600').isLeapYear():  true
    moment('1800').isLeapYear():  false
    moment('2000').isLeapYear():  true
    moment('2020').isLeapYear():  true
```

### 1.5 格式化输出

最后一个是时间的格式化输出，是我们可以直接透过口令(模版)来进行字符串格式化输出，而不再需要自己组合时间对象的属性

```js
moment#format(template)  // 根据模版(口令)进行格式化并返回字符串
moment#from(other)  // 输出自 other 到当前实例的语义化时间间隔
moment#diff(other)  // 比较与 other 相差的时间，返回时间戳
moment#unix()       // 当前时间的 unix 时间戳(单位：秒)
moment#valueOf()    // 当前时间的时间戳(单位：毫秒)
```

- `src/moment/5_format.js`

```js
const moment = require('moment')
const { log, group } = require('../utils/group')

group('moment.js format', () => {
  let earlyTime = moment([2020, 1, 1])
  const origin = moment()
  let time = origin.clone()

  log(`earlyTime:  ${earlyTime}`)
  log(`time:       ${time}`)

  group('format', () => {
    log(`time.format():                       ${time.format()}`)
    log(`time.format('YYYY-MM-DD'):           ${time.format('YYYY-MM-DD')}`)
    log(`time.format('YYYY.MM.DD HH:mm:ss'):  ${time.format('YYYY.MM.DD HH:mm:ss')}`)
  })

  group('from', () => {
    log(`time.from(earlyTime):        ${time.from(earlyTime)}`)
    log(`time.from(earlyTime, true):  ${time.from(earlyTime, true)}`)
  })

  group('diff', () => {
    log(`time.subtract(earlyTime).valueOf():  ${time.subtract(earlyTime).valueOf()}`)
    time = origin.clone()
    log(`time.diff(earlyTime):                ${time.diff(earlyTime)}`)
    log(`typeof time.subtract(earlyTime):     ${typeof time.subtract(earlyTime)}`)
    log(`typeof time.diff(earlyTime):         ${typeof time.diff(earlyTime)}`)
  })

  group('unix/valueOf', () => {
    time = origin.clone()
    log(`time.subtract(earlyTime).unix():     ${time.subtract(earlyTime).unix()}`)
    time = origin.clone()
    log(`time.subtract(earlyTime).valueOf():  ${time.subtract(earlyTime).valueOf()}`)
  })
})
```

- 输出

```
moment.js format
  earlyTime:  Sat Feb 01 2020 00:00:00 GMT+0800
  time:       Fri May 21 2021 14:37:10 GMT+0800
  format
    time.format():                       2021-05-21T14:37:10+08:00
    time.format('YYYY-MM-DD'):           2021-05-21
    time.format('YYYY.MM.DD HH:mm:ss'):  2021.05.21 14:37:10
  from
    time.from(earlyTime):        in a year
    time.from(earlyTime, true):  a year
  diff
    time.subtract(earlyTime).valueOf():  41092630034
    time.diff(earlyTime):                41092630034
    typeof time.subtract(earlyTime):     object
    typeof time.diff(earlyTime):         number
  unix/valueOf
    time.subtract(earlyTime).unix():     41092630
    time.subtract(earlyTime).valueOf():  41092630034
✨  Done in 0.19s.
```

## 2. Day.js

Day.js 的能力几乎与 Moment.js 一模一样，最主要的差别在于模块化方式和实现细节，下一个段落会进行单独说明。

下面我们先来看看直接从 moment 复制过来的示例代码，你会发现根本就是 99.99% 相似

### 2.1 创建时间日期对象

- `src/dayjs/1_parse.js`

```js
const dayjs = require('dayjs')
const customParseFormat = require('dayjs/plugin/customParseFormat')
const { log, group } = require('../utils/group')

group('day.js parse', () => {
  dayjs.extend(customParseFormat)
  group('dayjs', () => {
    log(`dayjs():                            ${dayjs().format()}`)
    log(`dayjs('2021-11-01'):                ${dayjs('2021-11-01').format()}`)
    log(`dayjs('2021-11-01 23:59:59'):       ${dayjs('2021-11-01 23:59:59').format()}`)
    log(`dayjs('20211101T235959'):           ${dayjs('20211101T235959').format()}`)
    log(`dayjs('01-11-2021', 'DD-MM-YYYY'):  ${dayjs('01-11-2021', 'DD-MM-YYYY').format()}`)
  })

  group('clone', () => {
    const m1 = dayjs()
    const m2 = dayjs(m1)
    const m3 = m2.clone()
    log(`m1:  ${m1.toString()}`)
    log(`m2:  ${m2.toString()}`)
    log(`m3:  ${m3.toString()}`)
    log(`m1 === m2:                        ${m1 === m2}`)
    log(`m1 === m3:                        ${m1 === m3}`)
    log(`m2 === m3:                        ${m2 === m3}`)
    log(`m1.toString() === m2.toString():  ${m1.toString() === m2.toString()}`)
    log(`m1.toString() === m3.toString():  ${m1.toString() === m3.toString()}`)
    log(`m2.toString() === m3.toString():  ${m2.toString() === m3.toString()}`)
  })
})
```

- 输出

```
day.js parse
  dayjs
    dayjs():                            2021-05-21T15:44:40+08:00
    dayjs('2021-11-01'):                2021-11-01T00:00:00+08:00
    dayjs('2021-11-01 23:59:59'):       2021-11-01T23:59:59+08:00
    dayjs('20211101T235959'):           2021-11-01T23:59:59+08:00
    dayjs('01-11-2021', 'DD-MM-YYYY'):  2021-11-01T00:00:00+08:00
  clone
    m1:  Fri, 21 May 2021 07:44:40 GMT
    m2:  Fri, 21 May 2021 07:44:40 GMT
    m3:  Fri, 21 May 2021 07:44:40 GMT
    m1 === m2:                        false
    m1 === m3:                        false
    m2 === m3:                        false
    m1.toString() === m2.toString():  true
    m1.toString() === m3.toString():  true
    m2.toString() === m3.toString():  true
```

#### 2.1.1 差异

我们可以看到差别在于从 `moment` 函数变成使用 `dayjs` 函数，同时不能使用数组来描述时间，这个倒是比较好解决

同时对于格式化输出的能力，我们需要透过插件(Plugin)的方式来引入

```js
const customParseFormat = require('dayjs/plugin/customParseFormat')

dayjs.extend(customParseFormat)
```

### 2.2 获取/设置属性

- `src/dayjs/2_get_set.js`

```js
const dayjs = require('dayjs')
const { log, group } = require('../utils/group')

group('day.js get/set', () => {
  const origin = dayjs('2021-11-01 23:59:59.699')

  let time = origin.clone()

  group('second', () => {
    log(`time:              ${time}`)
    log(`time.second():     ${time.second()}`)
    log(`time.second(996):  ${time.second(30)}`)
    log(`time.second():     ${time.second()}`)
  })

  group('hour', () => {
    log(`time:          ${time}`)
    log(`time.hour():   ${time.hour()}`)
    log(`time.hour(1):  ${time.hour(1)}`)
    log(`time.hour():   ${time.hour()}`)
  })

  time = origin.clone()

  group('get/set', () => {
    log(`current time:             ${time}`)
    log(`time.get('year'):         ${time.get('year')}`)
    log(`time.get('month'):        ${time.get('month')}`)
    log(`time.get('date'):         ${time.get('date')}`)
    log(`time.get('day'):          ${time.get('day')}`)
    log(`time.get('hour'):         ${time.get('hour')}`)
    log(`time.get('minute'):       ${time.get('minute')}`)
    log(`time.get('second'):       ${time.get('second')}`)
    log(`time.get('millisecond'):  ${time.get('millisecond')}`)

    log(`time.set('year', 2022):      ${time = time.set('year', 2022)}`)
    log(`time.set('month', 0):        ${time = time.set('month', 0)}`)
    log(`time.set('date', 13):        ${time = time.set('date', 13)}`)
    log(`time.set('day', 1):          ${time = time.set('day', 1)}`)
    log(`time.set('hour', 1):         ${time = time.set('hour', 1)}`)
    log(`time.set('minute', 3):       ${time = time.set('minute', 3)}`)
    log(`time.set('second', 5):       ${time = time.set('second', 5)}`)
    log(`time.set('millisecond', 7):  ${time = time.set('millisecond', 7)}`)
  })
})
```

- 输出

```
day.js get/set
  second
    time:              Mon, 01 Nov 2021 15:59:59 GMT
    time.second():     59
    time.second(996):  Mon, 01 Nov 2021 15:59:30 GMT
    time.second():     59
  hour
    time:          Mon, 01 Nov 2021 15:59:59 GMT
    time.hour():   23
    time.hour(1):  Sun, 31 Oct 2021 17:59:59 GMT
    time.hour():   23
  get/set
    current time:             Mon, 01 Nov 2021 15:59:59 GMT
    time.get('year'):         2021
    time.get('month'):        10
    time.get('date'):         1
    time.get('day'):          1
    time.get('hour'):         23
    time.get('minute'):       59
    time.get('second'):       59
    time.get('millisecond'):  699
    time.set('year', 2022):      Tue, 01 Nov 2022 15:59:59 GMT
    time.set('month', 0):        Sat, 01 Jan 2022 15:59:59 GMT
    time.set('date', 13):        Thu, 13 Jan 2022 15:59:59 GMT
    time.set('day', 1):          Mon, 10 Jan 2022 15:59:59 GMT
    time.set('hour', 1):         Sun, 09 Jan 2022 17:59:59 GMT
    time.set('minute', 3):       Sun, 09 Jan 2022 17:03:59 GMT
    time.set('second', 5):       Sun, 09 Jan 2022 17:03:05 GMT
    time.set('millisecond', 7):  Sun, 09 Jan 2022 17:03:05 GMT
```

#### 2.2.1 差异

get/set 操作方法则一模一样，不同的是在 dayjs 里面的时间对象是一个不可改变，任何操作都会返回一个新的对象，而通常这也是比较好的表现，这便是比较需要注意的

### 2.3 日期操作(加、减、对齐、选择)

- `src/dayjs/3_operate.js`

```js
const dayjs = require('dayjs')
const { log, group } = require('../utils/group')
const minMax = require('dayjs/plugin/minMax')

dayjs.extend(minMax)

group('day.js operate', () => {
  const origin = dayjs('2019-11-01 23:59:59.699')

  let time = origin.clone()
  group('add', () => {
    log(`current time:            ${time}`)
    log(`time.add(365, 'days'):   ${time.add(365, 'days')}`)
    log(`time.add(1, 'years'):    ${time.add(1, 'years')}`)
    log(`time.add(8, 'd'):        ${time.add(8, 'd')}`)
    log(`time.add({ days: 7 }):   ${time.add({ days: 7 })}`)
  })

  time = origin.clone().year(2020)
  group('subtract', () => {
    log(`current time:                ${time}`)
    log(`time.subtract(365, 'days'):  ${time.subtract(365, 'days')}`)
    log(`time.subtract(1, 'years'):   ${time.subtract(1, 'years')}`)
  })

  group('startOf/endOf', () => {
    time = dayjs('2019-01-01 01:30:30.333')
    log(`time:                    ${time}`)
    log(`time.startOf('minute'):  ${time.clone().startOf('minute')}`)
    log(`time.startOf('hour'):    ${time.clone().startOf('hour')}`)

    log(`time:                  ${time}`)
    log(`time.endOf('minute'):  ${time.clone().endOf('minute')}`)
    log(`time.endOf('hour'):    ${time.clone().endOf('hour')}`)
  })

  group('max/min', () => {
    const laterOne = dayjs('2020-01-01 12')
    const earlyOne = dayjs('2020-01-01 05')
    log(`earlyOne:                                    ${earlyOne}`)
    log(`laterOne:                                    ${laterOne}`)

    log(`dayjs.max(earlyOne, laterOne):               ${dayjs.max(earlyOne, laterOne)}`)
    log(`dayjs.max(earlyOne, laterOne) === laterOne:  ${dayjs.max(earlyOne, laterOne) === laterOne}`)

    log(`dayjs.min(earlyOne, laterOne):               ${dayjs.min(earlyOne, laterOne)}`)
    log(`dayjs.min(earlyOne, laterOne) === earlyOne:  ${dayjs.min(earlyOne, laterOne) === earlyOne}`)
  })

})
```

- 输出

```
day.js operate
  add
    current time:            Fri, 01 Nov 2019 15:59:59 GMT
    time.add(365, 'days'):   Sat, 31 Oct 2020 15:59:59 GMT
    time.add(1, 'years'):    Sun, 01 Nov 2020 15:59:59 GMT
    time.add(8, 'd'):        Sat, 09 Nov 2019 15:59:59 GMT
    time.add({ days: 7 }):   Invalid Date
  subtract
    current time:                Sun, 01 Nov 2020 15:59:59 GMT
    time.subtract(365, 'days'):  Sat, 02 Nov 2019 15:59:59 GMT
    time.subtract(1, 'years'):   Fri, 01 Nov 2019 15:59:59 GMT
  startOf/endOf
    time:                    Mon, 31 Dec 2018 17:30:30 GMT
    time.startOf('minute'):  Mon, 31 Dec 2018 17:30:00 GMT
    time.startOf('hour'):    Mon, 31 Dec 2018 17:00:00 GMT
    time:                  Mon, 31 Dec 2018 17:30:30 GMT
    time.endOf('minute'):  Mon, 31 Dec 2018 17:30:59 GMT
    time.endOf('hour'):    Mon, 31 Dec 2018 17:59:59 GMT
  max/min
    earlyOne:                                    Tue, 31 Dec 2019 21:00:00 GMT
    laterOne:                                    Wed, 01 Jan 2020 04:00:00 GMT
    dayjs.max(earlyOne, laterOne):               Wed, 01 Jan 2020 04:00:00 GMT
    dayjs.max(earlyOne, laterOne) === laterOne:  true
    dayjs.min(earlyOne, laterOne):               Tue, 31 Dec 2019 21:00:00 GMT
    dayjs.min(earlyOne, laterOne) === earlyOne:  true
```

#### 2.3.1 差异

第三个对于时间类型的操作也是近乎一致；不同的是比大小的 `min/max` 方法一样要透过插件的形式来引入

```js
const minMax = require('dayjs/plugin/minMax')

dayjs.extend(minMax)
```

### 2.4 查询(判断、比较)

- `src/dayjs/4_query.js`

```js
const dayjs = require('dayjs')
const { log, group } = require('../utils/group')
const isBetween = require('dayjs/plugin/isBetween')
const isLeapYear = require('dayjs/plugin/isLeapYear')

dayjs.extend(isBetween)
dayjs.extend(isLeapYear)

group('day.js query', () => {
  const earlyOne = dayjs('2020-01-01 05')
  const laterOne = dayjs('2020-01-01 12')
  const middleOne = dayjs('2020-01-01 08')

  log(`earlyOne:   ${earlyOne}`)
  log(`laterOne:   ${laterOne}`)
  log(`middleOne:  ${middleOne}`)

  group('isBefore', () => {
    log(`earlyOne.isBefore(laterOne):  ${earlyOne.isBefore(laterOne)}`)
    log(`laterOne.isBefore(earlyOne):  ${laterOne.isBefore(earlyOne)}`)
    log(`earlyOne.isBefore(earlyOne):  ${earlyOne.isBefore(earlyOne)}`)
  })

  group('isSame', () => {
    log(`earlyOne.isSame(laterOne):  ${earlyOne.isSame(laterOne)}`)
    log(`laterOne.isSame(earlyOne):  ${laterOne.isSame(earlyOne)}`)
    log(`earlyOne.isSame(earlyOne):  ${earlyOne.isSame(earlyOne)}`)
  })

  group('isAfter', () => {
    log(`earlyOne.isAfter(laterOne):  ${earlyOne.isAfter(laterOne)}`)
    log(`laterOne.isAfter(earlyOne):  ${laterOne.isAfter(earlyOne)}`)
    log(`earlyOne.isAfter(earlyOne):  ${earlyOne.isAfter(earlyOne)}`)
  })

  group('isBetween', () => {
    log(`middleOne.isBetween(earlyOne, laterOne):  ${middleOne.isBetween(earlyOne, laterOne)}`)
    log(`middleOne.isBetween(laterOne, earlyOne):  ${middleOne.isBetween(laterOne, earlyOne)}`)
  })

  group('isLeapYear', () => {
    log(`moment('1600').isLeapYear():  ${dayjs('1600').isLeapYear()}`)
    log(`moment('1800').isLeapYear():  ${dayjs('1800').isLeapYear()}`)
    log(`moment('2000').isLeapYear():  ${dayjs('2000').isLeapYear()}`)
    log(`moment('2020').isLeapYear():  ${dayjs('2020').isLeapYear()}`)
  })
})
```

- 输出

```
day.js query
  earlyOne:   Tue, 31 Dec 2019 21:00:00 GMT
  laterOne:   Wed, 01 Jan 2020 04:00:00 GMT
  middleOne:  Wed, 01 Jan 2020 00:00:00 GMT
  isBefore
    earlyOne.isBefore(laterOne):  true
    laterOne.isBefore(earlyOne):  false
    earlyOne.isBefore(earlyOne):  false
  isSame
    earlyOne.isSame(laterOne):  false
    laterOne.isSame(earlyOne):  false
    earlyOne.isSame(earlyOne):  true
  isAfter
    earlyOne.isAfter(laterOne):  false
    laterOne.isAfter(earlyOne):  true
    earlyOne.isAfter(earlyOne):  false
  isBetween
    middleOne.isBetween(earlyOne, laterOne):  true
    middleOne.isBetween(laterOne, earlyOne):  true
  isLeapYear
    dayjs('1600').isLeapYear():  true
    dayjs('1800').isLeapYear():  false
    dayjs('2000').isLeapYear():  true
    dayjs('2020').isLeapYear():  true
```

#### 2.4.1 差异

同样的 `isBetween, isLeapYear` 需要使用插件引入

```js
const isBetween = require('dayjs/plugin/isBetween')
const isLeapYear = require('dayjs/plugin/isLeapYear')

dayjs.extend(isBetween)
dayjs.extend(isLeapYear)
```

### 2.5 格式化输出

- `src/dayjs/5_format.js`

```js
const dayjs = require('dayjs')
const { log, group } = require('../utils/group')
const relativeTime = require('dayjs/plugin/relativeTime')

dayjs.extend(relativeTime)

group('day.js format', () => {
  let earlyTime = dayjs([2020, 1, 1])
  const origin = dayjs()
  let time = origin.clone()

  log(`earlyTime:  ${earlyTime}`)
  log(`time:       ${time}`)

  group('format', () => {
    log(`time.format():                       ${time.format()}`)
    log(`time.format('YYYY-MM-DD'):           ${time.format('YYYY-MM-DD')}`)
    log(`time.format('YYYY.MM.DD HH:mm:ss'):  ${time.format('YYYY.MM.DD HH:mm:ss')}`)
  })

  group('from', () => {
    log(`time.from(earlyTime):        ${time.from(earlyTime)}`)
    log(`time.from(earlyTime, true):  ${time.from(earlyTime, true)}`)
  })

  group('diff', () => {
    log(`time.subtract(earlyTime).valueOf():  ${time.subtract(earlyTime).valueOf()}`)
    time = origin.clone()
    log(`time.diff(earlyTime):                ${time.diff(earlyTime)}`)
    log(`typeof time.subtract(earlyTime):     ${typeof time.subtract(earlyTime)}`)
    log(`typeof time.diff(earlyTime):         ${typeof time.diff(earlyTime)}`)
  })

  group('unix/valueOf', () => {
    time = origin.clone()
    log(`time.subtract(earlyTime).unix():     ${time.subtract(earlyTime).unix()}`)
    time = origin.clone()
    log(`time.subtract(earlyTime).valueOf():  ${time.subtract(earlyTime).valueOf()}`)
  })
})
```

- 输出

```
day.js format
  earlyTime:  Tue, 31 Dec 2019 16:00:00 GMT
  time:       Fri, 21 May 2021 07:44:40 GMT
  format
    time.format():                       2021-05-21T15:44:40+08:00
    time.format('YYYY-MM-DD'):           2021-05-21
    time.format('YYYY.MM.DD HH:mm:ss'):  2021.05.21 15:44:40
  from
    time.from(earlyTime):        in a year
    time.from(earlyTime, true):  a year
  diff
    time.subtract(earlyTime).valueOf():  43775080504
    time.diff(earlyTime):                43775080504
    typeof time.subtract(earlyTime):     object
    typeof time.diff(earlyTime):         number
  unix/valueOf
    time.subtract(earlyTime).unix():     43775080
    time.subtract(earlyTime).valueOf():  43775080504
✨  Done in 0.24s.
```

#### 2.5.1 差异

`from` 方法或其他相对时间的相关方法则需要使用 `relativeTime` 引入

```js
const relativeTime = require('dayjs/plugin/relativeTime')

dayjs.extend(relativeTime)
```

## 3. Day.js 相较于 Moment.js 的好处

接下来比较重要的是来说明一下为什么说用 Day.js 比 moment.js 好，首先我们先来说说 moment.js

moment.js 算是比较早期，同时经过多次版本的更新，不断有新功能被添加到整个 moment 对象上，当所有功能、特性都聚集在这一个 moment 对象的时候就显得异常臃肿，同时也代表着所有代码都关联在一起。而且很重要的一点是 moment.js 对于原生 Date 对象存在侵入性的修改，与 day.js 全部都迁移到自己的 dayjs 对象内部不同。

### 3.1 Moment.js 的问题

也就是说当我们使用 moment.js 的时候会存在以下问题：

1. 类库中代码非常 **臃肿**，携带非常多没有被使用到的多余代码
2. **多余的特性、检查**，使得因为一些没有被使用的特性，却总是需要在运行时花费额外开销来处理特性分类的问题
3. moment.js 自己的作者也声明由于 moment 对象已经变得过于臃肿，所以已经停止进行新版本新特性的更新，进入 **维护期**；同时作者也推荐使用其他更新的库如 Day.js，或是未来的新标准 `Intl` 对象的使用

### 3.2 Day.js 的好处

使用 Day.js 则会为我们带来以下好处

1. Day.js 的 **API 几乎与 Moment.js 有 9 成以上相似**，所以在进行迁移的时候的学习成本极低
2. Day.js 将不同的特性分成各个插件(Plugin)，使用者可以根据需要主动引入并对全局对象进行扩展

    ```js
    const xxx = require('dayjs/plugin/xxx')

    dayjs.extend(xxx)
    ```

    这对于我们来说不仅是代码上的鲁棒性更高，同时减少 dayjs 对象的复杂度和大小。同时这对于我们的项目进行打包的时候也有助于打包工具的 **tree-shaking** 可以显著的降低无用代码和无用特性的引入。

# 结语

本篇提供了两个日期时间库使用时的大量示例，供大家参考和查阅，更详细更精确的 API 使用推荐大家去看类库的官方文档边查边用更对味。

# 其他资源

## 参考连接

| Title                                   | Link                                                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| moment.js                               | [http://momentjs.cn/](http://momentjs.cn/)                                                                   |
| day.js                                  | [https://dayjs.fenxianglu.cn/](https://dayjs.fenxianglu.cn/)                                                 |
| 时间标准基础知识UTC和ISO8601            | [https://www.cnblogs.com/doit8791/p/10398997.html](https://www.cnblogs.com/doit8791/p/10398997.html)         |
| Moment.js官方推荐使用其它时间处理库代替 | [https://www.w3cschool.cn/article/b0266c23d158f0.html](https://www.w3cschool.cn/article/b0266c23d158f0.html) |
|                                         | []()                                                                                                         |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/moment-dayjs](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/moment-dayjs)
