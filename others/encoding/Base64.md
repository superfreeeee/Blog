# Base64 & Base64URL 编码方案(附 js 代码实现)

@[TOC](文章目录)

<!-- TOC -->

- [Base64 & Base64URL 编码方案(附 js 代码实现)](#base64--base64url-编码方案附-js-代码实现)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [什么是 Base64？](#什么是-base64)
  - [Base64 转换规则](#base64-转换规则)
    - [Base64URL 规则](#base64url-规则)
    - [Base64 转换字符表](#base64-转换字符表)
    - [转换规则图解](#转换规则图解)
  - [在 JavaScript 中进行 Base64 编码](#在-javascript-中进行-base64-编码)
    - [Base64 编码 js 实现](#base64-编码-js-实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

本篇将会介绍 Base64 编码方案的规则、用途等，最后附上一段 js 代码的 Base64 编码解码实现。

## 参考

<table>
  <tr>
    <td>base64-百度百科</td>
    <td><a href="https://baike.baidu.com/item/base64/8545775">https://baike.baidu.com/item/base64/8545775</a></td>
  </tr>
  <tr>
    <td>Base64和Base64URL</td>
    <td><a href="https://blog.csdn.net/weixin_42506905/article/details/82053888">https://blog.csdn.net/weixin_42506905/article/details/82053888</a></td>
  </tr>
  <tr>
    <td>如何检查字符串是否为base64编码</td>
    <td><a href="https://www.imooc.com/wenda/detail/578611">https://www.imooc.com/wenda/detail/578611</a></td>
  </tr>
  <tr>
    <td>dankogai/js-base64</td>
    <td><a href="https://github.com/dankogai/js-base64/tree/375a99b0d85b0c03133924b2a1135d0ca2d11247">https://github.com/dankogai/js-base64/tree/375a99b0d85b0c03133924b2a1135d0ca2d11247</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/others/encoding/base64">https://github.com/superfreeeee/Blog-code/tree/main/others/encoding/base64</a>

# 正文

## 什么是 Base64？

Base64 是一种编码方案，它能够将二进制流转换为可打印字符，基本思想就是透过二进制数据的截取方式将 0~255 的 ASCII 码映射为 0~63 组成的可打印字符。

基于这种非常直接的转换格式，相当于是对二进制流进行了一次对称明文加密方案，同时编码后的字符流也能更直接的反应信息长度等信息。

Base64 主要的作用在 Http 协议中将二进制流转换为可打印字符流进行传输，前端也有根据 Base64 编码来代表二进制文件、图片的应用。

## Base64 转换规则

前面说到 Base64 是对二进制流进行编码的方案，Base64 将任意 **二进制字节流** 以三个字节为一组，共 24 bits 拆分成 4 个 6bits 的单元，再将新的 6bits 单元映射成 64 个可打印字符即完成编码。然而以三个字符为一组最后可能会剩余 1 个或 2 个字符，则需要进行额外补全。下面列出最终编码时的三条规则：

1. 整个二进制流从开始以 **三个字节** 为一组，共 24bits 拆分为 **4 个 6bits** 并映射到 **64 个可打印字符**，若存在剩余 1~2 个字节不成组，则采用 2、3 点的规则进行处理。

2. 若最后剩余 2 个字节：第 3 个字节以 0 填充，编码后的最后 **1 个字符以 '=' 填充**

3. 若最后剩余 1 个字节：第 2、3 个字节以 0 填充，编码后的最后 **2 个字符以 '=' 填充**

### Base64URL 规则

Base64 能够将二进制流很好的转换为一串可打印字符流，然而编码后的 `=`、`/`、`+` 等字符不利于 url 中的查询参数、数据库保存时的转义等，所以在实际应用的场景中又产生了一种几乎等价的编码方案 **Base64URL**。

Base64URL 的基本规则与 Base64 一模一样，差别在于最后两个特殊字符 `+`、`/` 改成使用  `-`、`_`，同时也去除末尾额外添加的 `=` 字符即完成 Base64URL 的编码

### Base64 转换字符表

重新分组后的 6bits 单元依据下表映射成 64 个可打印字符

![](https://picures.oss-cn-beijing.aliyuncs.com/img/base64_alphabet.png)

### 转换规则图解

下面给出 123、12、1 三种情况的编码结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/base64_transform.png)

## 在 JavaScript 中进行 Base64 编码

在 JavaScript 中 Base64 是经常会使用到的编码方式，我们也不想重复造轮子，因此我们可以透过下面几种方法进行 Base64 编码

- 浏览器 window 对象下的 `btoa`、`atob` 进行编码、解码
- 第三方库 Base64.js (整合兼容性)
- canvas.toDataURL

### Base64 编码 js 实现

下面我们给出一个 js 实现的版本，参照 Base64.js 实现

- Base64.js

```js
const chars =
  'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/='
const charsArr = chars.split('')
const charsMap = {}
charsArr.forEach((c, i) => (charsMap[c] = i))

const encode = (s) => {
  const mod = s.length % 3
  let c1, c2, c3, u24, b1, b2, b3, b4, res
  res = ''
  for (let i = 0; i < s.length; ) {
    c1 = s.charCodeAt(i++)
    c2 = s.charCodeAt(i++)
    c3 = s.charCodeAt(i++)
    u24 = (c1 << 16) | (c2 << 8) | c3
    b1 = charsArr[(u24 >> 18) & 0x3f]
    b2 = charsArr[(u24 >> 12) & 0x3f]
    b3 = charsArr[(u24 >> 6) & 0x3f]
    b4 = charsArr[u24 & 0x3f]
    res += `${b1}${b2}${b3}${b4}`
  }
  return mod ? res.slice(0, mod - 3) + '==='.substring(mod) : res
}

const decode = (s) => {
  let b1, b2, b3, b4, u24, res, i
  s = s.replace(/=/g, '')
  const mod = s.length % 4
  const end = s.length - mod
  res = ''
  const _fromCC = String.fromCharCode.bind(String)
  for (i = 0; i < end; ) {
    b1 = charsMap[s.charAt(i++)]
    b2 = charsMap[s.charAt(i++)]
    b3 = charsMap[s.charAt(i++)] & 0x3f
    b4 = charsMap[s.charAt(i++)] & 0x3f
    u24 = (b1 << 18) | (b2 << 12) | (b3 << 6) | b4
    c1 = _fromCC((u24 >>> 16) & 0xff)
    c2 = _fromCC((u24 >>> 8) & 0xff)
    c3 = _fromCC(u24 & 0xff)
    res += `${c1}${c2}${c3}`
  }
  if (mod === 2) {
    b1 = charsMap[s.charAt(i++)]
    b2 = charsMap[s.charAt(i++)]
    c1 = _fromCC((b1 << 2) | (b2 >> 4))
    res += c1
  } else if (mod === 3) {
    b1 = charsMap[s.charAt(i++)]
    b2 = charsMap[s.charAt(i++)]
    b3 = charsMap[s.charAt(i++)]
    u24 = (b1 << 10) | (b2 << 4) | (b3 >> 2)
    c1 = _fromCC(u24 >> 8)
    c2 = _fromCC(u24 & 0xff)
    res += `${c1}${c2}`
  }
  return res
}

const isBase64 = (s) => {
  const chars = '[A-Za-z0-9+/]'
  const pattern = new RegExp(`^(${chars}{4})*(${chars}{3}=|${chars}{2}==)?$`)
  // ^([A-Za-z0-9+/]{4})*([A-Za-z0-9+/]{3}=|[A-Za-z0-9+/]{2}==)?$
  return pattern.test(s)
}

module.exports = { encode, decode, isBase64 }
```

- Base64.test.js

```js
const { test, expect } = require('@jest/globals')
// const { encode, decode, isBase64 } = require('../src/base64')
const base64 = require('../src/base64')

const tests = {
  encode: [
    { s: '123', res: 'MTIz' },
    { s: '123456', res: 'MTIzNDU2' },
    { s: '123\\n456', res: 'MTIzXG40NTY=' },
    { s: '123\n456', res: 'MTIzCjQ1Ng==' },
    {
      s:
        '2345678ijhgfdsq234567ikjhgdsw4567uikjn\nbde567uikjnbfdr6uiolkmnbvdsw4567890plkf\nrtiop[][-098uytrew3456yujkl;[][-098ytrewsdfgh',
      res:
        'MjM0NTY3OGlqaGdmZHNxMjM0NTY3aWtqaGdkc3c0NTY3dWlram4KYmRlNTY3dWlram5iZmRyNnVpb2xrbW5idmRzdzQ1Njc4OTBwbGtmCnJ0aW9wW11bLTA5OHV5dHJldzM0NTZ5dWprbDtbXVstMDk4eXRyZXdzZGZnaA==',
    },
  ],

  decode: [
    { s: 'MTIz', res: '123' },
    { s: 'MTIzNDU2', res: '123456' },
    { s: 'MTIzXG40NTY=', res: '123\\n456' },
    { s: 'MTIzCjQ1Ng==', res: '123\n456' },
    {
      s:
        'MjM0NTY3OGlqaGdmZHNxMjM0NTY3aWtqaGdkc3c0NTY3dWlram4KYmRlNTY3dWlram5iZmRyNnVpb2xrbW5idmRzdzQ1Njc4OTBwbGtmCnJ0aW9wW11bLTA5OHV5dHJldzM0NTZ5dWprbDtbXVstMDk4eXRyZXdzZGZnaA==',
      res:
        '2345678ijhgfdsq234567ikjhgdsw4567uikjn\nbde567uikjnbfdr6uiolkmnbvdsw4567890plkf\nrtiop[][-098uytrew3456yujkl;[][-098ytrewsdfgh',
    },
  ],
  isBase64: [
    { s: 'MTIz', res: true },
    { s: 'MTIzNDU2', res: true },
    { s: 'MTIzCjQ1Ng==', res: true },
    { s: '', res: true },
    {
      s:
        'MWU1Njc4aW9sa2poZ2ZkZXJ1aWtuYnZmZHI2dWlramhnZmRlMzQ1cmV3YXp4Y3Zibm1sb2l1NzY1cmVzZGN2Ym5qawp5dWlrbmJ2Y3hzd2VydHl1aW9rbmJ2Y2RzZXJ0eXVpb2xrbW5idmZkZXJ0eXVpa21uYnZmZHI=',
      res: true,
    },
    { s: 'MTIz??', res: false },
    { s: '???MTIz', res: false },
    { s: 'MTIz!!%^())*&*^%$#@@@', res: false },
    {
      s:
        'MWU1Njc4aW9sa2poZ2ZkZXJ1aWtuYnZmZHI2dWlramhnZmRlMzQ1cmV3YXp4Y3Zibm1sb2l1NzY1cmVzZGN2Ym5qawp5dWlrbmJ2Y3hzd2VydHl1aW9rbmJ2Y2RzZXJ0eXVpb2xrbW5idmZkZXJ0eXVpa21uYnZmZHI!',
      res: false,
    },
  ],
}

Reflect.ownKeys(tests).forEach((name) => {
  tests[name].forEach(({ s, res }, index) => {
    test(`test ${name} ${index + 1}`, () => {
      expect(base64[name](s)).toBe(res)
    })
  })
})
```

# 结语

Base64 编码虽然是基本算是明文几乎没有加密的作用，但是作为一个特殊的编码规范，在许多地方都被广泛使用，编码后的字母空间也变得足够小，在传输时对二进制编码进行编码之后就能够像普通的字符流一样传输，比起 16 进制来说又剩下不少空间，供大家参考。
