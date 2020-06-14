# Vue：Vuex 狀態管理

@[TOC](文章目錄)

<!-- TOC -->

- [Vue：Vuex 狀態管理](#vuevuex-狀態管理)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Install 安裝](#install-安裝)
    - [CDN](#cdn)
    - [NPM](#npm)
  - [Architecture 總體架構](#architecture-總體架構)
  - [State](#state)
    - [mapState](#mapstate)
  - [Getters](#getters)
    - [透過方法訪問](#透過方法訪問)
    - [mapGetters](#mapgetters)
  - [Mutations](#mutations)
    - [常量方法名](#常量方法名)
    - [注意事項](#注意事項)
    - [mapMutations](#mapmutations)
  - [Actions](#actions)
    - [調用異步 API](#調用異步-api)
    - [返回值處理](#返回值處理)
    - [async/await](#asyncawait)
    - [組合 Action 函數](#組合-action-函數)
    - [mapActions](#mapactions)
  - [Module](#module)
    - [局部狀態](#局部狀態)
      - [Getters & Mutations](#getters--mutations)
      - [Actions](#actions-1)
    - [命名空間](#命名空間)
    - [帶模塊 Action 註冊到全局](#帶模塊-action-註冊到全局)
    - [帶命名空間的綁定函數](#帶命名空間的綁定函數)
  - [Usage 使用](#usage-使用)
    - [Plugin 插件](#plugin-插件)
    - [strict 嚴格模式](#strict-嚴格模式)
    - [Form 表單處理](#form-表單處理)
- [結語](#結語)
  - [Quick Start](#quick-start)

<!-- /TOC -->

## 簡介

首先聲明本篇介紹 Vuex 較為詳細，如果希望快速掌握使用重點的可以查詢最後的<a href="#Quick Start">Quick Start</a>

本篇來介紹專為 Vue 開發的狀態管理模式：Vuex。如同 React 擁有 Redux 一樣，Vue 也自己開發了一款狀態管理機制 Vuex，如果你不打算開發大型的單面應用，我們可以使用簡易版的<a href="https://cn.vuejs.org/v2/guide/state-management.html">store 模式</a>來應對。

由於 Vue、React 等響應式框架都是單向數據流模式，所以通常情況下我們可以很輕易的將多個子組件的狀態提升到共同父組件管理。然而當兩個組件距離太過麻煩，甚至需要子組件往父組件傳遞狀態的時候就會異常的麻煩。因此引入了一個`管理共享狀態`的思想。獨立出一個全局數據池，並且掛載在 Vue 實例上，使得所有組件都能夠訪問並修改。接下來就來看看 Vuex 的用法吧。

## 參考

<table>
  <tr>
    <td>Vuex-官方</td>
    <td><a href="https://vuex.vuejs.org/zh/">https://vuex.vuejs.org/zh/</a></td>
  </tr>
</table>

# 正文

## Install 安裝

任何庫都是從安裝開始

### CDN

直接下載並透過 `<script>` 標籤引用：

```js
// link: https://unpkg.com/vuex
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>
```

### NPM

```bash
$ npm i -S vuex
```

使用模塊化打包工具時需要使用 `Vue.use` 裡顯示安裝：

- main.js

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

## Architecture 總體架構

我們先來看看 Vuex 的總體架構（參考官方例圖）：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vuex_architecture.png)

整體架構有以下幾個主要節點

1. State：管理全局共享狀態
2. Getters：State 的衍伸狀態，或是單純的別名(也可以用 `mapState`)
3. Mutations：真正且唯一修改狀態的方法，並且必須是`同步操作`
4. Actions：負責組合多個狀態的更新，並且管理所有`異步操作`
5. Components：根據 State 選染組件，修改狀態時提交更新(`dispatch/commit`)

代碼架構如下：

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {},
  getters: {},
  mutations: {},
  actions: {},
  getters: {},
  modules: {}
})
```

具體項目創建方式可以參考之前寫的：<a href="https://blog.csdn.net/weixin_44691608/article/details/105879105">Vue 項目啟動</a>，並在選擇插件的時候將 Vuex 選起來，開箱即用！

接下來我們來介紹各個部件的用法

## State

Vuex 使用單一狀態樹，也就是說一個應用同時將只會存在一個 store 實例，用一個對象包含所有層級的狀態。Vuex 會遞歸的將 data 上的所有 properties 轉化為 getter/setter(Vue2)，因此觀察後就不能再添加新的屬性，因此推薦在初始化時先將所有根級狀態定義好：

- /store/index.js

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    userInfo: {
      id: 0,
      name: 'John'
    }
  }
  // ...
})
```

創建好 store 對象之後註冊到 Vue 實例上：

- main.js

```js
import Vue from 'vue'
import App from './App.vue'
import store from './store'

Vue.config.productionTip = false

new Vue({
  store,
  render: (h) => h(App)
}).$mount('#app')
```

最後在組件引用的時候透過 `computed` 作為計算屬性引入：

- App.vue

```html
<template>
  <div id="app">
    {{ userInfo }}
  </div>
</template>

<script>
  export default {
    computed: {
      userInfo() {
        return this.$store.state.userInfo
      }
    }
  }
</script>
```

store 是存在於全局的狀態管理對象，可以看成一個全局的狀態池，而所有組建都能夠從狀態池中提取(作為 `computed` 計算屬性引入)狀態。

### mapState

除了在計算屬性內部透過 `this.$store.state.xxx` 提取狀態之外，我們還可以透過 `mapState` 輔助函數直接引入多個狀態並自動生成計算屬性：

```js
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 屬性值使用函數，接收 state 為唯一參數
    count: (state) => state.count,

    // 直接串入字符串，等同於 state => state.xxx
    countAlias: 'count',

    // 使用一般函數寫法以訪問組件實例(this)
    countWithLocal(state) {
      return state.count + this.localCount
    }
  })
}
```

如果屬性名與 state 內部相同不變，可以直接傳入字符串數組：

```js
computed: mapState([
  // this.count 將訪問 store.state.count
  'count'
])
```

最終版本，透過展開運算符(`...`)將 State 狀態混入計算屬性中：

```js
computed: {
  ...mapeState([
    'count'
  ]),
  localProp() {
    // ...
  }
}
```

## Getters

我們現在能夠透過計算屬性以及 `mapState` 將狀態引入到組件，然而我們可能會想提供一些派生屬性，如下：

```js
computed: {
  doneTodoList() {
    return this.$store.state.todoList.filter(todo => todo.done)
  }
}
```

我們並不希望在每個組建要用到的時候都重新建立這個計算屬性，而是向 store 提取屬性的時候能夠直接提取到這個派生屬性，同時也能夠避免同樣的計算屬性重複定義：

- /store/index.js

```js
const store = new Vuex.Store({
  state: {
    todoList: [
      { id: 0, text: 'todo case 1', done: false },
      { id: 1, text: 'todo case 2', done: true },
      { id: 2, text: 'todo case 3', done: true },
      { id: 3, text: 'todo case 4', done: false },
      { id: 4, text: 'todo case 5', done: true }
    ]
  },
  getters: {
    doneTodoList: (state) => state.todoList.filter((todo) => todo.done),
    // 第二個參數為 getters，可以引用其他派生屬性
    doneTodoCount: (state, getters) => getters.doneTodoList.length,
    firstDoneTodo: (state, getters) => getters.doneTodoList[0]
  }
  // ...
})
```

- App.vue

```js
computed: {
  doneTodoCount() {
    return this.$store.getters.doneTodoCount
  },
  doneTodoList() {
    return this.$store.getters.doneTodoList
  }
}
```

### 透過方法訪問

getters 還有一個很有用技巧是返回一個查詢方法如下：

```js
const store = new Vuex.Store({
  state: {
    todoList: [
      { id: 0, text: 'todo case 1', done: false },
      { id: 1, text: 'todo case 2', done: true },
      { id: 2, text: 'todo case 3', done: true },
      { id: 3, text: 'todo case 4', done: false },
      { id: 4, text: 'todo case 5', done: true }
    ]
  },
  getters: {
    getTodoById: (state) => (id) =>
      state.todoList.find((todo) => todo.id === id)
  }
  // ...
})
```

這樣我們只需要引入一個派生屬性就能夠根據參數查詢多個屬性值：

```js
store.getters.getTodoById(2)
// { id: 2, text: 'todo case 3', done: true }
```

這邊用到了`柯里化(currying)`的思想，getters 計算屬性先綁定了 state 狀態，再接收使用者傳入的參數最後才返回真正的派生屬性

### mapGetters

與 `mapState` 函數相似，getters 也有自己的輔助函數 `mapGetters`，用法基本上與 `mapState` 一樣（後面的 `mapMutations`、`mapActions` 也都差不多）：

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    ...mapGetters([
      'doneTodoList',
      'doneTodoCount'
      // ...
    ])
  }
}
```

也可以取別名來避免命名空間衝突

```js
computed: {
  ...mapGetters({
    // this.doneCount 將指向 this.$store.getters.doneTodoCount
    doneCount: 'doneTodoCount'
  })
}
```

## Mutations

有了 State 和 Getters，我們可以保存一些全局狀態，並且提供一些根據 state 的計算屬性，接下來我們來介紹如何更改狀態。

由於 Vuex 負責狀態的維護和觀察，所以在嚴格模式下我們不可以直接修改 state 中的狀態，唯一修改狀態的方法就是提交 Mutation。我們需要先在 store 裡面定義 Mutation 函數（**注意**！必須是同步方法）：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    // 基本款
    increment(state) {
      state.count++
    },

    // Mutation 還最多可以接收一個參數作為載荷(payload)，傳入對象可作為多個參數
    setCount(state, newState) {
      state.count = newState.count
    }
  }
})
```

然後組建使用的時候需要透過`提交(commit)`來調用 Mutation 函數：

```js
// 基本款
store.commit('increment')

// 傳入載荷，也就是最多可以有一個參數
store.commit('setCount', { count: 0 })

// 使用對象風格的提交方式，Mutation 函數透過 type 參數傳遞
store.commit({
  type: 'increment'
})
store.commit({
  type: 'setCount',
  count: 0
})
```

### 常量方法名

有必要的話可以為 Mutations 函數建立常量函數名：

- /store/types

```js
export const mutationTypes = {
  m1: Symbol(),
  m2: Symbol()
}
```

- /store/index.js

```js
import Vuex from 'vuex'
import { mutationTypes } from './types'

const store = new Vuex.Store({
  state: {
    // ...
  },
  mutations: {
    [mutationTypes.m1]() {
      // ...
    },
    [mutationTypes.m2]() {
      // ...
    }
  }
})
```

- App.vue

```js
import { mutationTypes } from '../store/types'

store.commit(mutationTypes.m1)
```

### 注意事項

由於 store 狀態是響應式的，所以我們必須小心處理我們修改狀態的方式來避免有變量沒有被觀察到：

1. 在 store 初始化時聲明好根級對象的屬性
2. 添加新屬性時應該使用 `Vue.set` 或是使用`展開運算符(...)`（推薦）
3. 由於存在 Mutation 日誌觀察追蹤和狀態修改的問題，必須確保 Mutation 函數是同步函數，以避免觀察追蹤記錄丟失以及狀態非同步更新等問題（強制）

```js
const store = new Vuex.Store({
  state: {
    count: 0,
    userInfo: {},
    todoList: []
  },
  mutations: {
    // 基本款
    increment(state) {
      state.count++
    },

    // 使用 Vue.set 添加新屬性
    addUserInfoProp(state, prop) {
      Vue.set(state.userInfo, prop.key, prop.value)
    },

    // 使用展開運算符
    updateUserInfo(state, userInfo) {
      state.userInfo = {
        ...state.userInfo,
        ...userInfo
      }
    }
  }
})
```

```js
// 基本款
store.commit('increment')
// 使用 Vue.set 添加新屬性
store.commit('addUserInfoProp', {
  key: 'abc',
  value: 123
})
// 使用展開運算符
store.commit('updateUserInfo', {
  id: 0,
  name: 'John'
})
```

- 注意：如果使用包含 `type` 屬性的載荷，傳入的參數也會有 `type` 屬性，應避免直接展開傳入對象

### mapMutations

與 `mapState`、`mapGetters` 類似，直接上代碼：

- App.vue

```js
import { mapMutations } from 'vuex'

export default {
  methods: {
    // 將 this.xxx 映射為 this.$store.commit(xxx)
    ...mapMutations(['increment', 'addUserInfoProp', 'updateUserInfo']),

    // 使用別名
    ...mapMutations({
      inc: 'increment',
      addProp: 'addUserInfoProp'
    })
  }
}
```

由於異步操作將造成順序不確定的問題，所以 Mutation 函數照慣例`必須都是同步事務`，異步操作則需要放到後面的 `Actions` 中處理

## Actions

使用 Vuex 的時候應該盡量保持 Mutation 函數的原子性(也就是同步)，而 `Actions` 將負責組合和處理異步操作的邏輯，並且透過`提交 Mutation(commit)`來修改狀態：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    increment(ctx) {
      ctx.commit('increment')
    }
  }
})
```

Action 函數將接受一個 context 對象作為第一個參數(**這邊需要強調並不是 store 對象本身**)，同時我們可以透過 ES6 的參數解構來提取我們需要的屬性：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    increment({ commit }) {
      commit('increment')
    }
  }
})
```

同樣 Action 函數也能夠接受一個載荷：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    },
    setCount(state, count) {
      state.count = count
    }
  },
  actions: {
    increment({ commit }) {
      commit('increment')
    },
    // 接受至多一個參數
    setCount({ commit }, newCount) {
      commit('setCount', newCount)
    }
  }
})
```

並且透過 `dispatch` 函數進行分發：

```js
// 基本款
store.dispatch('increment')
// 接受載荷
store.dispatch('setCount', 0)
// 對象風格
store.dispatch({
  type: 'setName',
  name: 'John'
})
```

### 調用異步 API

與 Mutation 函數不同的是，Action 函數可以執行異步操作，以及控制分發多重 Mutation 的時機：

```js
actions: {
  getTodoList({ commit, state }, userId) {
    commit(types.LOAD_TODOLIST)

    getTodoListAPI(userId)
      .then(res => {
        commit(types.GET_TODOLIST_SUCCESS, res)
      })
      .catch(err => {
        commit(types.GET_TODOLIST_FAIL)
      })
  }
}
```

### 返回值處理

由於 Actions 函數是異步操作，所以其返回值會是一個 Promise 對象，組件分發 `dispatch` 的時候需要用 then 來處理返回值

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions({
      loginAct: 'login'
    }),
    login() {
      const loginForm = {
        // ...
      }
      this.loginAct(loginForm).then((success) => {
        if (success) {
          console.log('login success')
        } else {
          console.log('login fail')
        }
      })
    }
  }
}
```

### async/await

由於 Action 函數同時又負責調用後端 API，因此可能需要同步化後端 API 的異步調用，除了使用 `Promise.prototype.then` 方法傳入回調之外，還可以使用 ES7 `async/await` 的語法糖直接將 Action 函數同步化

```js
actions: {
  login: async ({ commit }, loginForm) => {
    const res = await loginAPI(loginForm)
    if (res && res.data.success) {
      const userInfo = res.data.content
      commit('setUserInfo', userInfo)
    }
    return res && res.data.success
  }
}
```

### 組合 Action 函數

也可以實現多個 Action 互相調用

```js
action: {
  actionA() {
    return actionB().then(res => res)
  },
  actionB() {
    return actionC().then(res => res)
  },
  actionC() {
    return xxxAPI()
  }
}
```

### mapActions

一樣的輔助函數，以下列出用法示例：

```js
import { mapActions } from 'vuex'

export default {
  methods: {
    // 將 this.xxx 映射為 this.$store.commit(xxx)
    ...mapActions(['getTodoList']),

    // 使用別名
    ...mapActions({
      pull: 'getTodoList'
    })
  }
}
```

## Module

當一個應用的所有狀態全部聚集到一個 state 裡面的時候就會顯得異常臃腫而且管理困難。這時候我們可以透過 `modules 模塊化`的方法來將整個應用的狀態切分成許多子模塊，每個模塊就像是一個小型的 store 實例：

```js
const moduleA = {
  state: {},
  mutations: {},
  actions: {}
}

const moduleB = {
  state: {},
  mutations: {},
  actions: {}
}

const moduleC = {
  state: {},
  mutations: {},
  actions: {}
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB,
    c: moduleC
  },
  state: {},
  mutations: {},
  actions: {}
})

export default store
```

### 局部狀態

#### Getters & Mutations

在子模塊的情況下，Getters 和 Mutations 函數所接收的第一個 state 參數為局部狀態，也就是當前模塊的 state：

```js
const moduleA = {
  state: {
    count: 0
  },
  mutations: {
    // 這裡的 state 是 moduleA.state 而不是全局狀態
    increment(state) {}
  },
  getters: {
    // 這裡的 state 也是局部的
    count: (state) => state.count
  }
}
```

同時對於 Getters 函數來說，根模塊的模塊將作為第三個參數暴露出來，而根模塊的 Getters 函數則作為第四個參數：

```js
const moduleA = {
  // ...
  getters: {
    // state 為此模塊的狀態，rootState 表示根模塊的狀態
    // getters 為本模塊的 getters，第四個參數 rootGetters 為根模塊的 getters
    rootCount(state, getters, rootState, rootGetters) {
      return rootState.count
    }
  }
}
```

#### Actions

而對於 Action 函數來說，根模塊的狀態將作為上下文的 `rootState` 屬性被暴露出來：

```js
const moduleA = {
  // ...
  actions: {
    getRootCount: ({ commit, rootState }) => {
      return rootState.count
    }
  }
}
```

### 命名空間

默認情況下，Getters、Mutations、Actions 函數都是註冊在`全局命名空間`之下的，也就是所有模塊都能夠直接調用，如果我們想區分不同命名空間的話可以設置 `namespace: true`：

```js
const store = new Vuex.Store({
  modules: {
    moduleA: {
      namespace: true,
      state: {
        count: 0
      },
      getters: {
        // 調用時：getters['moduleA/count']
        count: (state) => state.count
      },
      mutations: {
        // 調用時：commit('moduleA/increment')
        increment(state) {
          state.count++
        }
      },
      actions: {
        increment: ({ commit }) => {
          // 調用此模塊的 increment，也就是 moduleA/increment
          commit('increment')
          // 調用根模塊的 increment
          commit('incremnet', null, { root: true })
        }
      }
    }
  }
})
```

### 帶模塊 Action 註冊到全局

如果我們使用了 `namespace: true` 註冊了子模塊命名空間，但是又想要將 Action 函數註冊到全局命名空間，這時候就可以使用 `root: true`：

```js
const store = new Vuex.Store({
  modules: {
    moduleA: {
      namespace: true,
      actions: {
        someActions: {
          root: true,
          handler({ commit }) {
            commit('increment', null, { root: true })
          }
        }
      }
    }
  }
})
```

### 帶命名空間的綁定函數

當我們使用模塊化分綁定函數(`mapState`、`mapGetters`、`mapMutations`、`mapActions`)時，寫法會顯得有些繁瑣：

```js
...mapMutations([
  'moduleA/moduleB/moduleC/getTodoList',
  'moduleA/moduleB/moduleC/doneTodoList',
  'moduleA/moduleB/moduleC/todoListCount'
])
```

這時候我們可以透過傳遞模塊路徑做第一個參數來簡化：

```js
...mapMutations('moduleA/moduleB/moduleC'[
  'getTodoList',
  'doneTodoList',
  'todoListCount'
])
```

也可以使用 `createNameSpacedHelpers` 創建綁定好的輔助函數：

```js
import { createNamespacedHelpers } from 'vuex'

const { mapMutations } = createNamespacedHelpers('moduleA/moduleB/moduleC')

export default {
  methods: {
    ...mapMutations(['getTodoList', 'doneTodoList', 'todoListCount'])
  }
}
```

## Usage 使用

### Plugin 插件

Vuex 還能夠接受註冊一些`插件(Plugin)`，如 Mutation 日誌等，在創建 store 實例時傳入即可：

```js
const myPlugin = (store) => {
  store.subscribe((mutation, state) => {
    // 紀錄每次的 Mutation 行為
  })
}

const store = new Vuex.Store({
  //
  plugins: [myPlugin]
})
```

插件能夠`生成 State 快照`，`Mutation 紀錄保存`等，Vuex 還提供了一些內置插件如 `createLogger` 等，詳情可以查閱官方 API。

### strict 嚴格模式

在 Store 實例中使用嚴格模式的好處是：確保 Mutation 函數為狀態更新的唯一手段，避免其他函數的副作用直接修改狀態

```js
const store = new Vuex.Store({
  strict: process.env.NODE_ENV !== 'production'
})
```

### Form 表單處理

由於上面提到了，所有狀態的修改和更新最好都透過 Mutation 函數，也就是說如果我們想要將表單內容綁定到 Vuex 的話，直接使用 `v-model` 綁定變量是不行的，我們有下列幾種方法：

1. 提交時才將內容同步到 store 全局狀態

2. 綁定 `@input` 或 `@change` 事件，在每次修改時都調用一次 `commit` 提交修改

```html
<input :value="message" @change="messageChange" />
```

```js
export default {
  // ...
  computed: {
    ...mapState(['message'])
  },
  methods: {
    ...mapMutations(['set_message']),
    messageChange(e) {
      this.set_message(e.target.value)
    }
  }
}
```

3. 雙向綁定計算屬性

```html
<input :value="message" />
```

```js
export default {
  // ...
  computed: {
    message: {
      get() {
        return this.$store.state.message
      },
      set(val) {
        this.$store.commit('set_message', val)
      }
    }
  }
}
```

# 結語

撒花！終於寫完了。本篇非常詳細的介紹了一次 Vuex 中不同部件的使用方法，更多細節可以查詢官方說明和官方提供的 API。下面提供一些使用方法重點和總結：

## Quick Start

1. `mapState`、`mapGetters` 為提取狀態，應展開到 `computed` 計算屬性中
2. `mapMutations`、`mapActions` 為方法，應展開到 `methods` 方法中
3. 展開時有三種寫法：

```js
// 枚舉要引入的狀態或方法
// 將 this.xxx 映射到 this.$store.(state|getters|mutations|actions).xxx
mapXXX(['a', 'b', 'c'])

// 創建別名，避免命名空間衝突
mapXXX({
  aAlias: (state) => state.a,
  bAlias: (state) => state.b,
  cAlias: (state) => state.c
})

// 使用一般函數寫法來與組件內容混合（不推薦，會複雜化引入）
mapGetters({
  mixinCount(state) {
    return state.count + this.count
  }
})
```

4. 在模塊對象加入 `namespace: true` 來區隔子模塊的命名空間

到此就全部完成啦，其實撇開許多複雜的配置和模塊的分割，Vuex 還是能夠花費很少的配置達到開箱即用的效果。
