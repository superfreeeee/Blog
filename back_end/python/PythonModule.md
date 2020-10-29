# Python: 模块化

@[TOC](文章目錄)

<!-- TOC -->

- [Python: 模块化](#python-模块化)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Import 引入方法](#import-引入方法)
  - [Import 路径](#import-路径)
  - [自定义模块](#自定义模块)
  - [屏蔽模块名：使用 `__init__.py`](#屏蔽模块名使用-__init__py)
    - [引入一个目录](#引入一个目录)
    - [将模块中的方法暴露到模块中](#将模块中的方法暴露到模块中)
    - [嵌套的模块](#嵌套的模块)
- [結語](#結語)

<!-- /TOC -->

## 簡介

今天来介绍 python 的模块化机制。当软件开发到达一定的规模之后进行模块化是必须的，而在 python 的模块化机制中最关键的就是 `__init__.py` 文件，相对于直接根据文件名引入方法，提供了屏蔽模块内部结构的特性。

## 參考

<table>
  <tr>
    <td>Python 模块-菜鸟教程</td>
    <td><a href="https://www.runoob.com/python/python-modules.html">https://www.runoob.com/python/python-modules.html</a></td>
  </tr>
  <tr>
    <td>__init__.py 提前暴露要调用的方法、模块</td>
    <td><a href="https://blog.csdn.net/yjinyyzyq/article/details/85317112">https://blog.csdn.net/yjinyyzyq/article/details/85317112</a></td>
  </tr>
</table>

# 正文

## Import 引入方法

在一般的 python 编程中，我们常常会引入一些第三方包(package)，如 `math`、`numpy` 等，而这些包存放的位置可能是本地的 pip 目录之下，也可能是在各个项目中使用 `venv` 包建立独立的包依赖虚拟环境，使用方法如下

- 语法

```py
# 引入指定模块（使用 as 指定别名）
import <module-name> [as <module-alias>]
# 引入指定方法
from <module-name> import <func-name> [as <func-alias>] [, ...<func-name> [as <func-alias>]]
```

- 实例

```py
import numpy as np
from math import sin

if __name__ == '__main__':
    a = np.arange(0, 10)
    print(a) 
    # [0 1 2 3 4 5 6 7 8 9]
    
    sin0 = sin(0)
    print(sin0) 
    # 0.0
```

## Import 路径

那这时候可能会有些人困在包路径的问题上。其实 import 语句查找包的路径就两个（可能实际环境会存在多个可选的路径，但是使用 venv 并且建立良好的模块化结构之后，主要管理包的路径就是下面这两个

1. 相对路径（当前文件所处模块）
2. 全局环境第三方依赖

## 自定义模块

现在我们可以在我们的项目中建立一个自定义的模块，结构如下

- 项目结构

```
/app
    main.py
    moduleA.py
```

- `moduleA.py`

```py
def funcA():
    print('invoke funcA in moduleA')
```

- `main.py`

两种种引入模式

```py
# 1. 只引入方法
from moduleA import funcA

if __name__ == '__main__':
    funcA() # invoke funcA in moduleA
    
```

```py
# 2. 引入整个模块
import moduleA

if __name__ == '__main__':
    # 需要加上模块名来找到方法
    moduleA.funcA() # invoke funcA in moduleA
```

## 屏蔽模块名：使用 `__init__.py`

上面透过文件名（文件为单位的模块）来引入模块中的方法，然而在复杂的项目结构中，我们可能希望屏蔽模块内部的文件名和目录结构，我们可以透过加入 `__init__.py` 文件来使一个目录转换成一个 python 模块

### 引入一个目录

上面的例子我们将文件视为一个模块来引入，而当我们引入的文件是一个 python 模块的时候，则会执行 `__init__.py` 文件并导出模块，下面我们看看加载的过程

- 项目结构

```py
/app
    main.py
    /moduleA
        __init__.py
```

- `moduleA/__init__.py`

```py
print('load moduleA')
```

- `main.py`

```py
import moduleA

if __name__ == '__main__':
    pass

# output:
# load moduleA
```

当我们使用 `main.py` 作为主入口时，`import moduleA` 会加载 `moduleA/__init__.py` 并执行

### 将模块中的方法暴露到模块中

现在我们可能在 `/moduleA` 中写了很多其他的方法和函数，结构如下：

- 项目结构

```py
/app
    main.py
    /moduleA
        __init__.py
        printer.py
```

- `moduleA/printer.py`

```py
def funcA():
    print('invoke printerA.funcA')
```

如果我们从模块外部引用时需要知道方法的具体文件和路径会非常的不便，而且会出现异常复杂的 import 路径，严重影响代码可读性，所以我们可以在模块的 `__init__.py` 将模块内部的方法暴露到目录之下

- `moduleA/__init__.py`

```py
# 前面加上 . 指定当前目录下的模块
from .printer import funcA

print('load moduleA')
```

这时候我们就可以直接从 `/moduleA` 目录找到并引入 `funcA` 方法了

- `main.py`

```py
from moduleA import funcA

if __name__ == '__main__':
    funcA()

# output:
# load moduleA  ; 引入模块时的输出
# invoke printerA.funcA  ; 调用 funcA 方法的输出
```

### 嵌套的模块

嵌套的模块逻辑也是相似的，只要掌握 import 语句查找模块的路径顺序：当前目录 -\> 全局依赖，那么不论嵌套多少层都不会搞混，举个例子：

- 项目结构

```py
/app
    main.py
    /moduleA
        __init__.py
        printer.py
        /sub_module
            __init__.py
            sub_printer.py
```

- `moduleA/sub_module/sub_printer.py`

```py
def subFuncA():
    print('invoke sub_printer.subFuncA')
```

- `moduleA/sub_module/__init__.py`

```py
from .sub_printer import * # 引入 subFuncA 方法

print('load sub_module success')
```

- `moduleA/printer.py`

```py
def funcA():
    print('invoke printerA.funcA')
```

- `moduleA/__init__.py`

```py
from .printer import * # 引入 funcA 方法
from .sub_module import * # 引入 subFuncA 方法

print('load moduleA success')
```

- `main.py`

```py
from moduleA import funcA, subFuncA

if __name__ == '__main__':
    funcA()
    subFuncA()

# output:
# load sub_module success
# load moduleA success
# invoke printerA.funcA
# invoke sub_printer.subFuncA
```

从输出我们就能够轻易的看出模块引入的顺序（注意 import 使用的路径，都是直接引入目录作为模块而不是指定的文件）

- 模块执行循序

```

1. from moduleA import funcA, subFuncA
   引入 moduleA 目录作为模块，执行 moduleA.__init__.py
    1.1 from .printer import *
        引入 printer (文件作为模块)中的所有方法
    1.2 from .sub_module import *
        引入 sub_module 中(目录作为模块)的所有方法，执行 sub_module/__init__.py
        1.2.1 from .sub_printer import *
              引入 sub_printer (文件作为模块)中的所有方法
        1.2.2 print('load sub_module success')
              模块 sub_module(目录作为模块)导出完成并输出
    1.3 print('load moduleA success')
        模块 moduleA(目录作为模块)导出完成并输出
2. funcA() & subFuncA()
   执行从 moduleA 找到的两个方法并输出
```

# 結語

以上就是 python 项目中的模块化使用方法，核心思想就是以下两点

- import 语句查找模块顺序

  1. 当前目录
  2. 全局环境依赖(包)

- 以目录为模块时加载 `__init__.py` 的内容

良好的项目结构和模块化设计能够大大降低开发的复杂度和提高项目的可阅读性。
