# JS Web API: NetworkInformation 网络状况 API

@[TOC](文章目录)

<!-- TOC -->

- [JS Web API: NetworkInformation 网络状况 API](#js-web-api-networkinformation-网络状况-api)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 相关 API](#1-相关-api)
  - [2. 代码实现](#2-代码实现)
  - [3. 最终效果](#3-最终效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 基本信息

今天要给大家讲解的是利用浏览器的 WebAPI 来获取当前用户的网络状况，然而实际上浏览器 API 提供的状态还是比较粗浅，而且大多都是估计值，同时兼容性并不是那么好，所以如测速下载等功能实际上还是依赖服务端对前端行为的响应进行检测还是比较准确的

## 1. 相关 API

网络状况的相关 API 主要集中在 `NetworkInformation` 对象上（从 `Navigator.connection` 属性上获得），以及 `Window` 对象上的 `onLine` 属性

| 属性                             | 含义                         | 类型    |
| -------------------------------- | ---------------------------- | ------- |
| Window.online                    | 是否连上网络                 | boolean |
| NetworkInformation.type          | 网络类型                     | string  |
| NetworkInformation.effectiveType | 网络类型                     | string  |
| NetworkInformation.downlink      | 下载速度                     | number  |
| NetworkInformation.downlinkMax   | 最大下载速度                 | number  |
| NetworkInformation.rtt           | 信号延迟                     | number  |
| NetworkInformation.saveData      | 当前设备是否开启节省流量开关 | boolean |

另外还有三个事件我们可以监听用于更新网络状态

| 事件                        |
| --------------------------- |
| Window.ononline             |
| Window.onoffline            |
| NetworkInformation.onchange |

具体 API 细节自己查：[传送门：NetworkInformation - MDN](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation)

## 2. 代码实现

接下来我们尝试着写点代码来看看效果

- `/src/components/NetworkConnection.tsx`

首先我们先定义接下来要保存的网络状态，除了前面提到的之外，我们另外保留一个最后更新时间

```ts
interface NetworkState {
  since?: Date;
  online?: boolean;
  type?: string;
  effectiveType?: string;
  downlink?: number;
  downlinkMax?: number;
  rtt?: number;
  saveData?: boolean;
}
```

接下来写两个方法用于获取状态快照

```ts
const getConnection = () => {
  const nav = window.navigator;
  if (!nav) return null;
  // @ts-ignore
  return nav.connection || nav.mozConnection || nav.webkitConnection;
};

const getNetworkState = (): NetworkState => {
  const connection = getConnection();
  if (!connection) {
    return {
      since: new Date(),
      online: navigator.onLine,
    };
  }
  const state: NetworkState = {
    since: new Date(),
    online: navigator.onLine,
    type: connection.type,
    effectiveType: connection.effectiveType,
    downlink: connection.downlink,
    downlinkMax: connection.downlinkMax,
    rtt: connection.rtt,
    saveData: connection.saveData,
  };

  console.table(state);

  return state;
};
```

最后我们选择在 mount 的时候初始化网络状况并挂载监听函数

```ts
  const [state, setState] = useState<NetworkState>({});

  useEffect(() => {
    // init state
    setState(getNetworkState());

    const onStateChange = () => {
      setState(getNetworkState());
    };

    const connection = getConnection();

    window.addEventListener('online', onStateChange);
    window.addEventListener('offline', onStateChange);
    connection?.addEventListener('change', onStateChange);
    return () => {
      window.removeEventListener('online', onStateChange);
      window.removeEventListener('offline', onStateChange);
      connection?.removeEventListener('change', onStateChange);
    };
  }, []);
```

## 3. 最终效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_network_information_result.png)

老实说个人觉得效果不太好，可能还是搞个实际的服务端来进行真正的网络交互会更实在，还能从浏览器 http 请求头获取更多信息

# 其他资源

## 参考连接

| Title                      | Link                                                                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| NetworkInformation - MDN   | [https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation) |
| JS中监听网络状况的常用方法 | [https://segmentfault.com/a/1190000023957246](https://segmentfault.com/a/1190000023957246)                                                 |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_network_information](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_network_information)
