# Node 实战: 从无到有创建一个自己专属的 CLI 脚手架

@[TOC](文章目录)

<!-- TOC -->

- [Node 实战: 从无到有创建一个自己专属的 CLI 脚手架](#node-实战-从无到有创建一个自己专属的-cli-脚手架)
- [前言](#前言)
- [正文](#正文)
  - [1. 相关工具介绍](#1-相关工具介绍)
    - [1.1 chalk 彩色输出(各种输出样式附加)](#11-chalk-彩色输出各种输出样式附加)
    - [1.2 commander 启动参数解析](#12-commander-启动参数解析)
    - [1.3 clear 清屏](#13-clear-清屏)
    - [1.4 figlet 文字图形生成](#14-figlet-文字图形生成)
    - [1.5 inquirer 交互式提问(配置选项设置)](#15-inquirer-交互式提问配置选项设置)
    - [1.6 ora 加载动画(转圈圈)](#16-ora-加载动画转圈圈)
  - [2. 正式开始](#2-正式开始)
    - [2.0 脚手架目标 & 主要实现步骤](#20-脚手架目标--主要实现步骤)
    - [2.1 配置阶段：项目入口](#21-配置阶段项目入口)
      - [2.1.1 package.json 配置](#211-packagejson-配置)
      - [2.1.2 全局命令脚本入口](#212-全局命令脚本入口)
      - [2.1.3 yarn link 模拟发布](#213-yarn-link-模拟发布)
    - [2.2 配置阶段：起始脚本入口](#22-配置阶段起始脚本入口)
    - [2.3 配置阶段：路径配置 config](#23-配置阶段路径配置-config)
    - [2.4 配置阶段：控制台初始化 init](#24-配置阶段控制台初始化-init)
    - [2.5 配置阶段：配置选项设置 asking](#25-配置阶段配置选项设置-asking)
    - [2.6 构建阶段：根据模版生成项目](#26-构建阶段根据模版生成项目)
      - [2.6.1 提供模版](#261-提供模版)
      - [2.6.2 生成 package.json](#262-生成-packagejson)
      - [2.6.3 复制模版内容](#263-复制模版内容)
      - [2.6.4 运行 yarn 安装依赖](#264-运行-yarn-安装依赖)
    - [2.7 构建阶段：生成使用导引](#27-构建阶段生成使用导引)
    - [2.8 运行项目](#28-运行项目)
    - [2.9 其他工具介绍](#29-其他工具介绍)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

随着前端越来越复杂，框架层出不穷，前端工程的配置也越来越繁重，所以衍生出了所谓的脚手架，实际上就是一种交互式的命令行工具，差别在于脚手架通常就是用于创建项目，简化依赖的配置、安装等活动。

本篇就带大家在 Node 环境下创建一个自己专属的 CLI 脚手架

# 正文

## 1. 相关工具介绍

在正式开始之前我们先熟悉一下待会创建 CLI 脚手架时会用到的一些三方库。

如果对这些库没有兴趣或是已经有一定程度的掌握的话，可以直接跳到第二节[正式开始](#2-正式开始)

### 1.1 chalk 彩色输出(各种输出样式附加)

chalk 用于产生命令行环境下的彩色输出

- 参考文档地址

[https://github.com/chalk/chalk](https://github.com/chalk/chalk)

- 安装依赖

```bash
$ yarn add chalk
```

- 代码示例 & 输出

```js
const chalk = require('chalk')

// 改变颜色
console.log(chalk.cyan('Color Text'))
console.log(chalk.bgCyan('Background Color Text'))

// 文字修饰
console.log(chalk.bold('Bold Text'))
console.log(chalk.italic('Italic Text'))
console.log(chalk.underline('Underline Text'))

console.log(chalk.dim('Dim Text'))
console.log(chalk.dim(chalk.cyan('Dim cyan Text')))
console.log(chalk.inverse('Inverse Text'))
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_chalk.png)

### 1.2 commander 启动参数解析

commander 主要用于处理用户的启动命令参数等

- 参考文档地址

[https://github.com/tj/commander.js](https://github.com/tj/commander.js)

- 安装依赖

```bash
$ yarn add commander
```

- 代码示例 & 输出

```js
#!/usr/bin/env node

const { program } = require('commander')

program
  .version('v1.0.0', '-v, --version')
  .usage('<command> <param1> [param2]')
  .command('hello <param1> [param2]')
  .action((param1, param2) => {
    console.log('param1', param1)
    console.log('param2', param2)
  })

program.parse()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_commander.png)

有关如何使用 Node 制作全局命令的部分放到后面

### 1.3 clear 清屏

clear 就比较简单了，其实就跟 `console.clear` 或是命令行下的 `clear` 一样，就是清屏

- 参考文档地址

[https://github.com/bahamas10/node-clear](https://github.com/bahamas10/node-clear)

- 安装依赖

```bash
$ yarn add clear
```

- 代码示例

```js
const clear = require('clear')
clear()
```

功能很简单，就不放动图了

### 1.4 figlet 文字图形生成

figlet 就比较有趣了，是用于生成文字图形，就如同 spring boot 或是 redis 那种字符合成的文字图形

- 参考文档地址

[https://github.com/patorjk/figlet.js](https://github.com/patorjk/figlet.js)

- 安装依赖

```bash
$ yarn add figlet
```

- 代码示例 & 输出

```js
const figlet = require('figlet')

figlet('Text 1', (err, text) => {
  console.log(text)
  console.log(figlet.textSync('Text 2', { font: 'Standard' }))
  console.log(figlet.textSync('Text 3', { font: 'Ghost' }))
  console.log(figlet.textSync('Text 4', { verticalLayout: 'fitted' }))
  console.log(figlet.textSync('Text 5', { verticalLayout: 'full' }))
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_figlet.png)

### 1.5 inquirer 交互式提问(配置选项设置)

commander 处理的是启动命令和参数解析等作用，而 inquirer 处理的则是启动后的选项配置处理，透过交互式问答的方式供用户选择需要的配置

- 参考文档地址

[https://github.com/SBoudrias/Inquirer.js](https://github.com/SBoudrias/Inquirer.js)

- 安装依赖

```bash
$ yarn add inquirer
```

- 代码示例 & 输出

inquirer 提供多种提问方法

1. 普通文本：**`type: input`**

```js
    {
      type: 'input',
      name: 'ans1',
      message: 'question1:',
      default: 'default ans1',
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer1.png)

2. 无编号列表(单选)：**`type: list`**

```js
    {
      type: 'list',
      name: 'ans2',
      message: 'question2(list)',
      choices: [
        '0',
        1,
        { name: 'options2', value: 2 },
        { name: 'option3', value: 3, short: 'op4' },
      ],
    },
```

choices 来配置选项

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer2.png)

3. 编号列表(单选)：**`type: rawlist`**

```js
    {
      type: 'rawlist',
      name: 'ans3',
      message: 'question3(rawlist)',
      choices: Array(5)
        .fill(0)
        .map((_, idx) => ({
          name: `op${idx}`,
          value: idx,
        })),
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer3.png)

4. 确认(是非题)：**`type: confirm`**

```js
    {
      type: 'confirm',
      name: 'ans4',
      message: 'question4(confirm)',
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer4.png)

5. 扩展列表(单选，自定义编号)：**`type: expand`**

```js
    {
      type: 'expand',
      name: 'ans5',
      message: 'question5(expand)',
      choices: [
        { name: 'op0', value: 0, key: 'a' },
        { name: 'op1', value: 1, key: 'b' },
        { name: 'op2', value: 2, key: 'c' },
        { name: 'op3', value: 3, key: 'd' },
      ],
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer5.png)

6. 多选框：**`type: checkbox`**

```js
    {
      type: 'checkbox',
      name: 'ans6',
      message: 'question6(checkbox)',
      choices: Array(5)
        .fill(0)
        .map((_, idx) => ({
          name: `op${idx}`,
          value: idx,
        })),
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer6.png)

7. 密码输入：**`type: password`**

```js
    {
      type: 'password',
      name: 'ans7',
      message: 'question7(password)',
    },
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer7.png)

8. 使用示例 & 输出

```js
const inquirer = require('inquirer')

inquirer
  .prompt([/* questions */])
  .then((answers) => {
    console.log('answers', answers)
  })
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_inquirer8.png)

### 1.6 ora 加载动画(转圈圈)

ora 提供一个动态的加载小圈圈，适合需要长时间任务时(如 `yarn install`)提升用户等待体验，同时配合文字帮助用户了解加载进度的功能

- 参考文档地址

[https://github.com/sindresorhus/ora](https://github.com/sindresorhus/ora)

- 安装依赖

```bash
$ yarn add ora
```

- 代码示例 & 输出

```js
const ora = require('ora')

const spinner = ora()

function test(tag, type, ms) {
  return new Promise((resolve, reject) => {
    spinner.start(tag)

    setTimeout(() => {
      spinner[type](`${tag} ${type}`)

      resolve()
    }, ms)
  })
}

test('test 1', 'succeed', 1000)
  .then(() => {
    return test('test 2', 'fail', 1000)
  })
  .then(() => {
    spinner.start('123')
    setTimeout(() => {
      spinner.text = '456'
      setTimeout(() => {
        spinner.text = '789'
        spinner.stopAndPersist()
      }, 1000)
    }, 1000)
  })
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_sample_ora.gif)

## 2. 正式开始

好了，相关工具的介绍已经完成了，相信大家已经迫不及待的要进入正式的脚手架创建环节了。

### 2.0 脚手架目标 & 主要实现步骤

首先我们要先明确一个目标：脚手架到底要拿来干嘛？主要有哪些步骤？

- 目标

    我们的目标就是创建出一个项目生成工具，能够根据用户的选项和参数来生成合适的前端项目

- 步骤

    本篇实现的脚手架步骤分成三大阶段

    1. 配置阶段：用户启动命令并传入参数，并完成配置选项设置
    1. 构建阶段：复制选项对应模版到目标目录下
    2. 安装阶段：`yarn install` 安装依赖

### 2.1 配置阶段：项目入口

首先是项目入口，我们当然不希望用个脚手架还要 `node xxx`，用起来也太麻烦。

所以这边来介绍一种使用 npm 创建全局参数的方式，就可以直接使用 `my-cli` 全局命令启动脚本了！

#### 2.1.1 package.json 配置

首先我们先看看 `package.json` 的配置

- `package.json`

```json
{
  "name": "node_cli",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "bin": {
    "my-cli": "bin/my-cli",
    "my-cli-create": "bin/my-cli-create",
    "my-cli-test-commander": "samples/commander.js"
  },
  "dependencies": {
    "chalk": "^4.1.1",
    "clear": "^0.1.0",
    "commander": "^7.2.0",
    "figlet": "^1.5.0",
    "inquirer": "^8.1.1",
    "ora": "^5.4.1",
    "prettier": "^2.3.2",
    "progress": "^2.0.3",
    "single-line-log": "^1.1.2"
  }
}
```

这里的重点在于 `bin` 选项，我们将预计要使用的全局命令作为键名，而脚本入口路径则写在值的部分(我们可以看到刚刚的 commander 测试命令也在其中)。

#### 2.1.2 全局命令脚本入口

接下来看到我们的主要脚本入口 `bin/my-cli`

- `/bin/my-cli`

```js
#!/usr/bin/env node

console.log('my-cli')
```

这种全局脚本与一般脚本不同的是，需要在开头写上 `#!/usr/bin/env node`，告诉系统使用 `node` 启动

#### 2.1.3 yarn link 模拟发布

最后我们使用 `yarn link` 命令，为该项目提供软连接，也就是类似全局安装的 npm 包一样。如此一来我们就能够使用 `my-cli` 全局命令启动并执行脚本了

### 2.2 配置阶段：起始脚本入口

接下来我们看到起始脚本的内容

- `/bin/my-cli`

```js
#!/usr/bin/env node

const { program } = require('commander')

program
  .version('v1.0.0', '-v, --version')
  .usage('<command> [options]')
  .command('create [projectName]', 'for creating new project')

program.parse()
```

这边我们使用 commander 的特性来处理命令提示，而我们在顶层命令 `my-cli` 下，再创建一个子命令 `create`，同时根据上面的配置，当用户输入 `my-cli create xxx` 的时候，就会自动寻找是否存在 `my-cli-create` 的命令，而根据前面 `package.json` 的配置，也就是 `bin/my-cli-create` 文件

- `/bin/my-cli-create`

```js
#!/usr/bin/env node

const init = require('../lib/init')
const asking = require('../lib/asking')
const build = require('../lib/build')
const post = require('../lib/post')
const { program } = require('commander')
const ensureTargetDir = require('../lib/ensureTargetDir')

program.parse()

const { projectName, templatesPath, targetDir } = ensureTargetDir(
  program.args[0]
)

console.log('templatesPath:', templatesPath)
console.log('targetDir    :', targetDir)

init()
  .then(() => {
    return asking(projectName)
  })
  .then((options) => {
    return build(options, { templatesPath, targetDir })
  })
  .then(() => {
    post(targetDir)
  })
  .catch((err) => {
    console.log('error occur')
    console.log(err)
  })
```

下面我们一一解释每一步骤的含义

### 2.3 配置阶段：路径配置 config

首先是第一阶段，要先根据用户的输入确定模版路径和目标目录路径

- `/bin/my-cli-create`

```js
// ...

const ensureTargetDir = require('../lib/ensureTargetDir')

const { projectName, templatesPath, targetDir } = ensureTargetDir(
  program.args[0]
)

console.log('templatesPath:', templatesPath)
console.log('targetDir    :', targetDir)

// ...
```

- `/lib/ensureTargetDir.js`

```js
const path = require('path')
const { readdirSync, mkdirSync } = require('fs')
const { templatesPath, currentPath } = require('../config')

const ensureTargetDir = (projectName) => {
  let targetDir
  if (!projectName || projectName === '.') {
    // my-cli create .
    if (readdirSync(currentPath).length > 0) {
      console.log('targetDir is not empty', readdirSync(currentPath))
      throw new Error('targetDir is not empty')
    }
    targetDir = currentPath
    projectName = path.basename(targetDir)
  } else {
    // my-cli create <projectName>
    targetDir = path.join(currentPath, projectName)
    mkdirSync(targetDir)
  }
  return {
    projectName,
    templatesPath,
    targetDir,
  }
}

module.exports = ensureTargetDir
```

- `config.js`

```js
const path = require('path')

const rootPath = __dirname
const templatesPath = path.join(rootPath, 'templates')
const currentPath = path.resolve('./')

module.exports = {
  rootPath,
  templatesPath,
  currentPath,
}
```

用户可以选择在当前目录下初始化项目(必须是空目录)，或是指定项目名称并创建新的目录作为项目的根目录

### 2.4 配置阶段：控制台初始化 init

接下来是全局的初始化

- `/lib/init.js`

```js
const figlet = require('figlet')
const clear = require('clear')
const chalk = require('chalk')

const init = async () => {
  clear()
  console.log(chalk.bold('starting my-cli v1.0.0'))
  const text = figlet.textSync('My Cli', { font: 'Ghost' })
  console.log(text)
}

module.exports = init
```

首先我们先清屏，然后利用 figlet 生成一个文字图标

### 2.5 配置阶段：配置选项设置 asking

接下来是透过 inquirer 来向用户提问

- `/lib/asking.js`

```js
const inquirer = require('inquirer')

const ask = async (questions) => {
  return await inquirer.prompt(questions)
}

const genQuestions = (projectName) => ({
  info: [
    {
      type: 'input',
      message: 'Project name:',
      name: 'name',
      default: projectName,
    },
    {
      type: 'input',
      name: 'author',
      default: 'superfree',
    },
    {
      type: 'list',
      name: 'type',
      choices: ['node', 'web'],
    },
  ],
  node: [
    {
      type: 'confirm',
      name: 'useBabel',
      message: 'Using Babel?',
      default: false,
    },
    {
      when: (answers) => {
        if (!answers.useBabel) {
          answers.useTS = false
          return false
        }
        return true
      },
      type: 'confirm',
      name: 'useTS',
      message: 'Using Typescript?',
      default: false,
    },
  ],
  web: [
    {
      type: 'confirm',
      name: 'useWebpack',
      message: 'Using Webpack?',
    },
  ],
})

const getTemplate = (type, options) => {
  if (type === 'node') {
    if (!options.useBabel) return 'node'
    if (!options.useTS) return 'node_babel'
    return 'node_babel_ts'
  } else if (type === 'web') {
    return 'web'
  } else {
    throw new Error('unkown type')
  }
}

const asking = async (projectName) => {
  const questions = genQuestions(projectName)
  try {
    const info = await ask(questions.info)
    // console.log('info', info)
    const options = await ask(questions[info.type])
    // console.log('options', options)
    const template = getTemplate(info.type, options)
    return {
      ...info,
      ...options,
      template,
    }
  } catch (err) {
    return Promise.reject(err)
  }
}

module.exports = asking
```

### 2.6 构建阶段：根据模版生成项目

用户确认好配置之后下一步就是 **构建阶段**，根据配置来复制模版内容到项目目录下

构建阶段代码如下

- `/lib/build.js`

```js
const unreadyTemplate = ['web']

const build = async (options, { templatesPath, targetDir }) => {
  console.log('Your options:', options)
  if (unreadyTemplate.includes(options.template)) {
    console.log(`template '${options.template}' is not ready yet`)
    return
  }
  const templateBase = path.join(templatesPath, options.template)

  copyPackageJson(templateBase, targetDir, options)

  copyRestFiles(templateBase, targetDir)

  await yarnHelper.install(targetDir)
}
```

#### 2.6.1 提供模版

所以首先我们要先提供一个模版，这边以 node + babel + typescript 的配置方案为例，项目目录结构如下

```tree
templates/node_babel_ts
├── babel.config.json
├── package.json
├── src
│   └── index.ts
└── tsconfig.json

1 directory, 4 files
```

- `package.json` 配置文件

```json
{
  "name": "node-babel-ts",
  "author": "superfree",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "babel-node src/index -x \".ts\"",
    "build": "rm -rf lib && babel src/ -d lib/ -x \".ts\"",
    "clean": "rm -rf lib"
  },
  "devDependencies": {
    "@babel/cli": "^7.14.5",
    "@babel/core": "^7.14.6",
    "@babel/node": "^7.14.5",
    "@babel/preset-env": "^7.14.5",
    "@babel/preset-typescript": "^7.14.5",
    "@types/node": "^16.0.0",
    "typescript": "^4.3.3"
  }
}
```

- `babel.config.json` 配置文件

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

- `tsconfig.json` 配置文件

```json
{
  "compilerOptions": {
    "target": "es5",
    "allowJs": true,
    "strict": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "experimentalDecorators": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

- `/src/index.ts` 项目入口

```ts
const s: string = 'Hello world'
console.log(s)
```

#### 2.6.2 生成 package.json

有了模版之后我们就可以复制模版内容并稍作修改，加入用户输入配置

- `/lib/build.js`

```js
// copy package.json
const copyPackageJson = (templateBase, targetDir, { name, author }) => {
  const packageJson = JSON.parse(
    readFileSync(path.join(templateBase, 'package.json'))
  )
  packageJson.name = name
  packageJson.author = author

  writeFileSync(
    path.join(targetDir, 'package.json'),
    prettier.format(JSON.stringify(packageJson), { parser: 'json-stringify' })
  )
}
```

#### 2.6.3 复制模版内容

主要就 package.json 需要特别配置，其他配置我们可以直接复制模版到目标目录就好了

- `/lib/build.js`

```js
// copy rest
const copyRestFiles = (baseDir, targetDir, excludes = ['package.json']) => {
  // get files
  const files = readdirSync(baseDir).filter(
    (fileName) => !excludes.includes(fileName)
  )
  files.forEach((fileName) => {
    const sourceFilePath = path.join(baseDir, fileName)
    const targetFilePath = path.join(targetDir, fileName)

    const stats = statSync(sourceFilePath)
    if (stats.isDirectory()) {
      // dir
      // ensure dir exists
      if (!existsSync(targetFilePath)) {
        mkdirSync(targetFilePath)
      }

      copyRestFiles(sourceFilePath, targetFilePath, excludes)
    } else {
      // plain files
      const fileContent = readFileSync(sourceFilePath)
      writeFileSync(targetFilePath, fileContent)
    }
  })
}
```

#### 2.6.4 运行 yarn 安装依赖

构建阶段最后一步则是执行 `yarn install` 命令安装依赖

- `/lib/yarn.js`

```js
const { exec, execSync } = require('child_process')

const spinner = require('./spinner')
const console = require('./console')

const install = (targetDir) => {
  return new Promise((resolve, reject) => {
    spinner.start('Start yarn install ...')

    exec('yarn', { cwd: targetDir }, (err, stdout, stderr) => {
      if (err) reject(err)
      spinner.success('yarn install success')
      console.log(stdout)
      resolve()
    })
  })
}

module.exports = {
  install,
}
```

### 2.7 构建阶段：生成使用导引

项目全部生成并安装完毕之后，我们可以再加一个使用导引，指引用户如何启动项目

- `/lib/post.js`

```js
const path = require('path')
const chalk = require('chalk')
const { currentPath } = require('../config')

const showAction = (target, cmd) => {
  console.log(`${target}, run:`)
  console.log(`    ${chalk.cyan(cmd)}`)
  console.log()
}

const post = (targetDir) => {
  if (targetDir !== currentPath) {
    const projectName = path.basename(targetDir)
    showAction('To move inside the project', `cd ${projectName}`)
  }
  showAction('Run project', `yarn start`)
  showAction('Build & package project', `yarn build`)
}

module.exports = post
```

### 2.8 运行项目

好了最终我们以 node + babel + ts 的配置为例，看看脚手架的使用与输出结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_cli_create_console.png)

### 2.9 其他工具介绍

除了本篇提到的很多工具之外，其实还有其他包也能够派上用上：

- Handlebars 模版编译
- MetalSmith 静态网站生成

上面是 @vue/cli 脚手架里面用到的 npm 包，其他可以自己去寻找一些能派上用场的包。

# 结语

到这里就完成了，作者创立脚手架的目的在于为自己提供一个用于创建轻量级可配置的小型测试项目的工具，同时在使用脚手架和配置项目的过程也是帮助自己多了解项目配置相关的细节，增加对于项目环境搭建的掌控性。

# 其他资源

## 参考连接

| Title                                                         | Link                                                                                                                                                                                         |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| chalk - Github                                                | [https://github.com/chalk/chalk](https://github.com/chalk/chalk)                                                                                                                             |
| commander.js - Github                                         | [https://github.com/tj/commander.js](https://github.com/tj/commander.js)                                                                                                                     |
| node-clear - Github                                           | [https://github.com/bahamas10/node-clear](https://github.com/bahamas10/node-clear)                                                                                                           |
| figlet.js - Github                                            | [https://github.com/patorjk/figlet.js](https://github.com/patorjk/figlet.js)                                                                                                                 |
| Inquirer - Github                                             | [https://github.com/SBoudrias/Inquirer.js](https://github.com/SBoudrias/Inquirer.js)                                                                                                         |
| ora - Github                                                  | [https://github.com/sindresorhus/ora](https://github.com/sindresorhus/ora)                                                                                                                   |
| prettier                                                      | [https://prettier.io/](https://prettier.io/)                                                                                                                                                 |
| handlebars                                                    | [https://handlebarsjs.com/zh/](https://handlebarsjs.com/zh/)                                                                                                                                 |
| MetalSmith                                                    | [https://metalsmith.io/](https://metalsmith.io/)                                                                                                                                             |
| vue-cli                                                       | [https://github.com/vuejs/vue-cli/tree/%40vue/cli%403.1.2](https://github.com/vuejs/vue-cli/tree/%40vue/cli%403.1.2)                                                                         |
| create-react-app                                              | [https://github.com/facebook/create-react-app](https://github.com/facebook/create-react-app)                                                                                                 |
| 前端保存之前输入的值_前端Node命令行交互工具——inquirer使用详解 | [https://blog.csdn.net/weixin_39638468/article/details/111370176](https://blog.csdn.net/weixin_39638468/article/details/111370176)                                                           |
| inquirer.js —— 一个用户与命令行交互的工具                     | [https://blog.csdn.net/qq_26733915/article/details/80461257?utm_source=app&app_version=4.9.2](https://blog.csdn.net/qq_26733915/article/details/80461257?utm_source=app&app_version=4.9.2)   |
| 教你用 Node 创建 CLI 工具                                     | [https://blog.csdn.net/qq_43327962/article/details/111062463?utm_source=app&app_version=4.9.2](https://blog.csdn.net/qq_43327962/article/details/111062463?utm_source=app&app_version=4.9.2) |
| node-npm发布包-package.json中bin的用法                        | [https://blog.csdn.net/weixin_43833570/article/details/97100520](https://blog.csdn.net/weixin_43833570/article/details/97100520)                                                             |
| node js学习 [第四篇] cli commander chalk Inquirer【新手向】   | [https://blog.csdn.net/qq_28008615/article/details/89954539](https://blog.csdn.net/qq_28008615/article/details/89954539)                                                                     |
| NodeJS获取当前目录和运行文件所在目录                          | [https://blog.csdn.net/mouday/article/details/105325635](https://blog.csdn.net/mouday/article/details/105325635)                                                                             |
| nodejs的execa库使用                                           | [http://abloz.com/tech/2018/08/21/nodejs-execa/](http://abloz.com/tech/2018/08/21/nodejs-execa/)                                                                                             |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/node/node_cli](https://github.com/superfreeeee/Blog-code/tree/main/front_end/node/node_cli)
