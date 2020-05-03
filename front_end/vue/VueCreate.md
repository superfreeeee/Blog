# Vue 項目啟動

<!-- TOC -->

- [Vue 項目啟動](#vue-項目啟動)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [CDN（不推薦）](#cdn不推薦)
        - [Sample](#sample)
    - [Vue-cli 官方腳手架（推薦）](#vue-cli-官方腳手架推薦)
        - [Installation 安裝](#installation-安裝)
        - [Terminal 透過命令行創建](#terminal-透過命令行創建)
        - [Vue UI 透過圖形化介面創建](#vue-ui-透過圖形化介面創建)
- [結語](#結語)

<!-- /TOC -->

## 簡介

有關 Vue 的基本介紹都可以在官網看到，由尤雨奚大大開發出來的響應式前端框架，除了官方推薦的輔助庫 vuex, vue-router，對於 webpack 和 babel 的配置也各有一套，同時在 kbone(有點爛), uni-app 等多端框架的集成更是成就非凡。本篇簡單介紹幾種基本 Vue 項目的創建方式。

## 參考

<table>
    <tr>
        <td>Vue 官網介紹</td>
        <td><a href="https://cn.vuejs.org/v2/guide/">https://cn.vuejs.org/v2/guide/</a></td>
    </tr>
</table>

# 正文

## CDN（不推薦）

透過在 head 標籤內引入 CDN(Content delivery network)，即可直接使用 Vue 的基本內容，是舊時代的開發方法，現在已經幾乎不太使用了

### Sample

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

## Vue-cli 官方腳手架（推薦）

vue-cli 除了簡單項目框架選項，有 babel, webpack, vuex, vue-router 等配置之外，同時還可以透過選項創建其他使用了 Vue 的框架模板項目，如 uni-app 等。

本篇僅介紹兩種基本 Vue 項目的創建流程

1. 一種是透過命令行使用 vue-cli
2. 一種使用 vue-cli 集成的圖形化介面

### Installation 安裝

透過 npm 在本地全局安裝 vue-cli（安裝 node 會自動安裝 npm 相關指令，window 需配置環境變量以全局使用 npm 命令）

```bash
$ npm i -g @vue/cli
```

### Terminal 透過命令行創建

1. 使用 `vue create` 命令啟動腳手架創建 vue 項目

```
$ vue create vue-demo
```

2. preset: 進入腳手架創建介面，可以選擇**默認配置**或是**手動配置**，選擇`手動配置`

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_1.png)

3. dependency: 接下來可以選擇配置項，可在此添加 typescript, vuex, vue-router 等依賴

- 操作：space 選擇，enter 完成選擇

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_2.png)

4. ESLint: 第三步結束後可能會接著一些依賴項的配置選項，本項目只選擇 Babel, Linter，需要配置 ESLint 的檢查規範程度。

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_3.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_4.png)

5. config: 其他開發時配置

- babel, eslint 配置位置

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_5.png)

- 保存配置模板(preset)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/terminal_6.png)

6. Finish: 創建完成後進入項目根目錄，使用 `npm run serve` 或 `npm run dev`啟動開發模式，項目啟動命令設置在 `package.json` 文件裡面配置

```bash
$ cd vue-demo
$ npm run serve
```

- Note: 相關開發環境配置，生產環境打包等說明在模板項目的 readme.md 裡面都有說明

### Vue UI 透過圖形化介面創建

1. 命令行輸入 `vue ui` 啟動 vue-cli 的圖形化介面

```bash
$ vue ui
```

2. 新增：選擇新增，選擇想要創建項目的指定目錄路徑，點選`+在此新增新專案`

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_1.png)

3. config: 配置項目名、炮管理器、git 倉庫初始化等選項。

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_2.png)

4. preset: 選擇模板，本次選擇手動配置

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_3.png)

5. dependency: 選擇項目使用依賴，本次選擇 Babel, ESLint

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_4.png)

6. setting: 依賴相關配置

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_5.png)

7. preset: 保存配置模板，選擇**不保存預設**

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_6.png)

8. dashboard: 創建完成後，進入項目儀表板

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_7.png)

9. develop: 選擇 任務 -> server -> 執行 -> 啟動 app，啟動開發環境

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/vueui_8.png)

# 結語

本篇演示兩種 Vue 項目的創建方式，都是透過 vue-cli 腳手架創建，腳手架詳細操作或其他擴展功能可上 Vue 官網查看細節。
