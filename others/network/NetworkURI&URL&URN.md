# 一次搞懂 URI、URL、URN

@[TOC](文章目录)

<!-- TOC -->

- [一次搞懂 URI、URL、URN](#一次搞懂-uriurlurn)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [名词解释](#名词解释)
  - [URI 格式](#uri-格式)
  - [URL 格式](#url-格式)
  - [URN 格式](#urn-格式)
- [结语](#结语)

<!-- /TOC -->

## 简介

在使用网络相关的 API 又或是信息传输、甚至微服务配置时常常看到 URI、URL 交错着出现，而与之相似的还有一个叫做 URN 的东西。平常最常接触到的就是 URL，通常在前后端的场景就把它当成请求的路由就算了，本篇就来细细品味这三个符号到底差在哪吧。

## 参考

<table>
  <tr>
    <td>統一資源標誌符-wikipedia</td>
    <td><a href="https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6">https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6</a></td>
  </tr>
  <tr>
    <td>統一資源定位符-wikipedia</td>
    <td><a href="https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6">https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6</a></td>
  </tr>
  <tr>
    <td>統一資源名稱-wikipedia</td>
    <td><a href="https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%90%8D%E7%A7%B0">https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%90%8D%E7%A7%B0</a></td>
  </tr>
  <tr>
    <td>URI-百度百科</td>
    <td><a href="https://baike.baidu.com/item/URI/2901761">https://baike.baidu.com/item/URI/2901761</a></td>
  </tr>
</table>

# 正文

## 名词解释

先来看看三个符号的说明

| 标识符 | 全称                                       | 作用             |
| ------ | ------------------------------------------ | ---------------- |
| URI    | Uniform Resource Identifier 统一资源标志符 | 唯一标识资源     |
| URL    | Uniform Resource Locator 统一资源定位符    | 唯一定位资源位置 |
| URN    | Uniform Resource Name 统一资源名称         | 唯一识别资源     |

从本质上来看三个符号都是用于表示`网络资源`，同时三个符号表示的范围还有如下的层级关系

![](https://picures.oss-cn-beijing.aliyuncs.com/img/uri&url&urn_layer.png)
(图自维基百科)

> 三者差异

URI 能`唯一表示资源`，URL 则是以`资源路径`来表示资源，URN 则是根据`名称（可能附带来源&ID）`来表示资源

好了再多的文字说明只会越来越混乱，我们都喜欢看一点实际的，接下来我们就来看看三个符号的通用格式

## URI 格式

首先先来看看 URI，根据维基百科，URI 通常表示成下列形式：

```
[协议名]://[用户名]:[密码]@[主机名]:[端口号]/[路径]?[查询参数]#[片段ID]
```

对其实本质上就跟 URL 几乎一样，相关历史缘故这边就不想说明了自己看参考链接hhh。然而 URI 可以说是 URL 和 URN 的超集，能表示的资源范围比 URL 广的许多，用 `is-a` 的关系来描述的话就是：URL、URL is-a URI

其实实际上使用的时候就只是名字问题，主要用得多的还是 URL 所以马上来看看下一个

## URL 格式

```
# 标准格式
[协议类型]://[服务器位置IP]:[端口]/[资源层级路径][资源名称]?[查询参数]#[片段ID]
# 完整格式
[协议类型]://[存取凭证]@[服务器位置IP]:[端口]/[资源层级路径][资源名称]?[查询参数]#[片段ID]
```

这时候骂娘的心都有了hhh，好先别激动，我知道根本就跟 URI 差不多，不过这就是我们最常用的 URL 格式完整表示，也就是平常我们拿来当作`网址`、或是一些`请求路由`，其实这两件事情本质上是一个动作：`请求网络资源`。前者可能是请求一个网页资源，后者可能请求服务器的计算资源并期待着返回计算结果等。

接下来看看常见的 url 例子

```
https://zh.wikipedia.org:443/w/index.php?title=Special:random
https                -> 协议类型
zh.wikipedia.org     -> 服务器位置（需要进行域名解析）
443                  -> 服务端口号
/w/index.php         -> 资源路径
title=Special:random -> 请求参数，形如 key=value 的键值对
```

## URN 格式

最后一个 URN 标识符，常见于论文索引，格式如下

```
urn:[NID]:[NSS]
```

- `NID` 命名空间：可以看作是资源的命名方式，例如 `isbn`、`isan`、`issn`、`ietf:rfc`、`mpeg:mpeg7:schema`
- `NSS` 序号：根据 `NID` 的不同代表不同的意义，如：
  - `isbn`：此时 NSS 表示`书号(book number)`
  - `isan`：此时 NSS 表示`媒体编号(audiovisual number)`
  - `issn`：此时 NSS 表示`序列号(serial number)`
  - `ietf:rfc`：此时 NSS 表示`IEFT RFC 论文`
  - `mpeg:mpeg7:schema`：此时 NSS 表示`MPEG-7 视屏类型`

好再多的自己查去吧，最后一样给几个例子hhh

| URN                                        | 说明                                                                                     |
| ------------------------------------------ | ---------------------------------------------------------------------------------------- |
| urn:isbn:0451450523                        | The 1968 book The Last Unicorn, identified by its book number.                           |
| urn:isan:0000-0000-9E59-0000-O-0000-0000-2 | The 2002 film Spider-Man, identified by its audiovisual number.                          |
| urn:issn:0167-6423                         | The scientific journal Science of Computer Programming, identified by its serial number. |
| urn:ietf:rfc:2648                          | The IETF's RFC 2648.                                                                     |
| urn:mpeg:mpeg7:schema:2001                 | The default namespace rules for MPEG-7 video metadata.                                   |
| urn:oid:2.16.840                           | The OID for the United States.                                                           |

# 结语

这样算是有解释到三个符号吧（参考了算是比较完整正式的解释），网上还有很多其他抽象或是比喻的解释，还是不太懂的就自己查一查吧hhh。
