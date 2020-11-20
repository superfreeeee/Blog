# Make 构建工具

@[TOC](文章目录)

<!-- TOC -->

- [Make 构建工具](#make-构建工具)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [Install 安装](#install-安装)
  - [make 初体验](#make-初体验)
  - [Makefile 结构](#makefile-结构)
    - [格式](#格式)
  - [Rule 构建规则](#rule-构建规则)
    - [规则一：默认目标](#规则一默认目标)
    - [规则二：构建条件](#规则二构建条件)
  - [Phony Target 伪目标](#phony-target-伪目标)
  - [Makefile 语法](#makefile-语法)
    - [回声 echoing：使用 `@` 取消](#回声-echoing使用--取消)
    - [注释 Comment：使用 `#`](#注释-comment使用-)
    - [变量声明：使用 `=` 声明、使用 `$()` 访问](#变量声明使用--声明使用--访问)
      - [Shell 变量：使用 `$$` 转译](#shell-变量使用--转译)
      - [共用 Shell：使用 `;`、`\`、`.ONESHELL`](#共用-shell使用-oneshell)
      - [内置变量](#内置变量)
      - [自动变量：`$@`、`$<`、`$^`、...](#自动变量)
    - [其他](#其他)
- [结语](#结语)

<!-- /TOC -->

## 简介

`make` 是 GNU 软件包中的一个`构建工具`，可用于构建并管理项目的目标文件和可执行文件，另外我们可以编写 `Makefile` 作为批处理命令的定义。由于属于 GNU 中的工具之一，所以 `make + Makefile` 的项目管理工具必须用于类 unix 环境，要想在 Windows 或其他环境使用就需要用到抽象程度更高的 cmake、qmake 等其他工具，这边就不做说明了。

## 参考

<table>
  <tr>
    <td>Make 命令教程-阮一峰</td>
    <td><a href="http://www.ruanyifeng.com/blog/2015/02/make.html">http://www.ruanyifeng.com/blog/2015/02/make.html</a></td>
  </tr>
  <tr>
    <td>make makefile cmake qmake都是什么，有什么区别？</td>
    <td><a href="https://www.zhihu.com/question/27455963">https://www.zhihu.com/question/27455963</a></td>
  </tr>
  <tr>
    <td>makefile内置变量</td>
    <td><a href="https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html">https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html</a></td>
  </tr>
  <tr>
    <td>makefile自动变量</td>
    <td><a href="https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html">https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html</a></td>
  </tr>
</table>

# 正文

## Install 安装

要使用一个工具当然要先安装

- Linux 环境下安装

```bash
# Ubuntu
$ apt-get install make

# CenOS
$ yun install make
```

- Mac 环境

```bash
$ xcode-select --install
```

## make 初体验

make 顾名思义就是`制作`的意思，命令默认会查找的当前目录下的 `Makefile` 作为批处理脚本，我们也可以使用 `-f` 参数指定使用脚本，咱们马上就来试试看吧（详细脚本结构和输出后面会进行说明）

> 项目结构

```tree
/
|- Makefile
|- a.txt
```

- `Makefile`

```makefile
all:
	echo "make something...(default"
```

- `a.txt`

```makefile
all:
	echo "make something...(by a.txt"
```

> 使用默认脚本(文件名 `Makefile`，大小写无关)

```bash
$ make
echo "make something...(default"
make something...(default
```

> 指定制作脚本

```bash
$ make -f a.txt
echo "make something...(by a.txt"
make something...(by a.txt
```

嗯输出了一些奇怪的东西，不过先不管，我们先来看看 makefile 里面都在写些啥呢。

## Makefile 结构

通常我们直接使用 `Makefile` 也就是默认的脚本文件名，不论写在哪，作为 make 构建的脚本内容格式是有规范的：

```makefile
<target>: [<dependency>, ...]
    <command>
    ...
```

前面说过 `make` 指令代表`制作`，也就是说我们进行构建时围绕的核心当然就是我们的`构建目标(target)`也就是我们的目标文件啦！而构建目标文件的时候可能会依赖一些其他已经存在的文件成为`依赖(dependency)`；最后如何构建，也就是具体的构建操作就由下面的`命令(command)序列`来指定。

看着上面的定义可能看不太懂，那就举个栗子：

> 示例

- `Makefile`

```makefile
main: main.c
    gcc main.c -o main
```

这个脚本的意思就是：

- `main` 作为构建目标
- `main.c` 作为构建依赖
- `gcc main.c -o main` 为实际构建时执行的指令

### 格式

由于 `Makefile` 作为批处理脚本，在写法上有些限制才能被工具正确解析

> 规则一：目标依赖间用空格隔开

```makefile
target: a.txt b.txt c.txt
# a.txt, b.txt, c.txt 三个文件名用空格隔开
```

> 规则二：命令前使用 tab 缩进

```makefile
init:
    echo "make"
# 注意！这里不可以使用四个空格代替 tab，必须要是真正的制表符 \t，否则会报错：*** missing separator.  Stop.
```

## Rule 构建规则

### 规则一：默认目标

我们有时候写好 Makefile 就不想每次都指定构建目标（反正每次都一样）或是想指定一个默认的构建目标。

在 make 指令的规则中，如果没有指定构建目标则会`默认构建第一个目标`（特殊目标不算）

> 默认构建目标

- `Makefile`

```makefile
default:
	echo "make default"
a:
	echo "make a"
b:
	echo "make b"
```

执行结果：

```bash
$ make a
echo "make a"
make a
$ make b
echo "make b"
make b
$ make
echo "make default"
make default
```

说明：这边我们定义了是三个目标，分别是 `default`、`a`、`b`，make 命令不指定目标是则会制作第一个目标也就是 `default`

### 规则二：构建条件

然而并不是所有时候 make 都会重新构建所有指定的目标的，只有在以下其中一项条件成立时才会进行重构建(否则保持原文件内容)

- 构建目标不存在
- 构建目标最后更新时间戳与依赖相同

在构建之前会检查`目标(target)`和`依赖(dependencies)`的`最后更新时间戳(latest update time)`，只有当时间戳不相同时，make 才会认为有必要重新进行构建

另外，如果依赖的文件也作为构建目标存在，make 原目标时将会优先进行依赖的构建，而后再进行当前目标的构建，举个栗子：

> 依赖同时也作为构建目标

- `Makefile`

```makefile
main: main.c
    gcc main.c -o main

main.c:
    echo "int main() {}" > main.c
```

执行结果：

```bash
$ make main
echo "make main.c"
make main.c
echo "int main() {}" > main.c
gcc main.c -o main
```

我们就会看到 `main.c` 和 `main` 都被制作出来了

## Phony Target 伪目标

有时候我们想要建立的目标不知一个或是我们只是单纯的想执行一个批处理操作，我们就可以使用`伪目标(phony target)`

其实伪目标和一般目标没什么区别，都是一个标识而已，而且也没规定说目标下的操作必须真的制作出目标文件，不过也有可能存在同名的文件的风险，所以我们需要借助特殊目标 `.PHONY` 的声明

> 使用 `.PHONY` 声明伪目标

- `Makefile`

```makefile
.PHONY: build clean

# 伪目标
build: main

clean:
	rm main.c main

# 目标文件
main: main.c
	gcc main.c -o main

# 源文件
main.c:
	echo "int main() {}" > main.c
```

执行结果：

```bash
$ make
echo "int main() {}" > main.c
gcc main.c -o main
$ make clean
rm main.c main
```

`.PHONY` 指定了 `build`、`clean` 作为伪目标，由于伪目标并不存在对应具体文件，所以每次都必定会被执行，相当于一段批处理脚本的命名。

执行流程：这里没有指定构建目标，所以默认找到的目标为 `build` 是一个伪目标，`build` 的依赖是 `main`，`main` 的依赖是 `main.c`，所以整个制作流程为：`main.c` -> `main`

## Makefile 语法

到此其实我们已经基本了解 make 的结构和规则了，最后介绍一些 `Makefile` 中的语法

### 回声 echoing：使用 `@` 取消

如果细心一些我们会发想上面示例中的输出都怪怪的，其实是因为 make 命令默认会打印每条`命令(command)`(这个行为称为`回声 echoing`)，而后才执行命令，我们可以在命令前加上 `@` 来取消回声

> 示例

- `Makefile`

```makefile
all: with_echo without_echo

with_echo:
	echo "make with_echo"

without_echo:
	@echo "make without_echo"
```

执行结果：

```bash
$ make
echo "make with_echo"
make with_echo
make without_echo
```

我们看到 `with_echo` 伪目标的命令被打印出来后才执行，而 `without_echo` 的则没有出现回声直接执行

### 注释 Comment：使用 `#`

在 Makefile 中使用 `#` 表示注释，回声同样会打印注释，需要注意

> 示例

- `Makefile`

```makefile
init:
	# 这是一个注释
	@# 取消回声的注释
	echo "make"
```

执行结果：

```bash
$ make
# 这是一个注释
echo "make"
make
```

### 变量声明：使用 `=` 声明、使用 `$()` 访问

同样在 Makefile 中我们可以声明变量，来一次操作多个目标（本质上就是`字符串直接替换`）

> 示例

- `Makefile`

```makefile
# 符号替换
CC = gcc

# 变量声明
SRC = main.c
BIN = main
ALL_OBJ = $(SRC) $(BIN)

context = "int main() {}"

# 目标
init: $(BIN)

clean:
	rm $(ALL_OBJ)

$(BIN): $(SRC)
	$(CC) $(SRC) -o $(BIN)

$(SRC):
	echo $(context) > $(SRC)
```

执行结果：

```bash
$ make
echo "int main() {}" > main.c
gcc main.c -o main
$ make clean
rm main.c main
```
#### Shell 变量：使用 `$$` 转译

注意：使用 Shell 变量时需要进行字符转译

> 使用 `$$` 转译使用 Shell 变量

- `Makefile`

```makefile
init:
	export STR="Hello World"; echo $$STR
```

执行结果：

```bash
$ make
export STR="Hello World"; echo $STR
Hello World
```

#### 共用 Shell：使用 `;`、`\`、`.ONESHELL`

由于每条`命令(command)`都是独立的 shell，所以要使用变量声明后引用必须使用 `;` 将多条命令合并成一行，或是使用 `\` 转译换行符，或是使用 `.ONESHELL` 指定共用 Shell

> 分别使用 `;`、`\`、`.ONESHELL` 实现使用同一个 SHELL

- `Makefile`

```makefile
a:
	export STR="Hello World"; echo $$STR

b:
	export STR="Hello World";\
	echo $$STR

.ONESHELL:
c:
	export STR="Hello World";
	echo $$STR
```

执行结果：

```bash
$ make a
export STR="Hello World"; echo $STR
Hello World
```

```bash
$ make b
export STR="Hello World";\
	echo $STR
Hello World
```

```bash
$ make c
export STR="Hello World";
echo $STR;
Hello World
```

#### 内置变量

make 命令还提供一些内置变量，以满足跨平台的兼容性，如 `$(CC)` 代表编译器，详细可以查看手册（[参考链接3](#参考)）

> 示例

- `Makefile`

```makefile
init:
	$(CC) -v
```

执行结果：

```bash
$ make
cc -v
Apple clang version 12.0.0 (clang-1200.0.32.21)
Target: x86_64-apple-darwin19.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```

说明：博主使用的环境为 mac 的 command-line-tools

#### 自动变量：`$@`、`$<`、`$^`、...

除了一般自定义变量和内置变量之外，我们还能够使用运行时自动替换的`自动变量`，常用下列几个，详细可以[参考链接4](#参考)：

| 自动变量 | 含义           |
| -------- | -------------- |
| `$@`     | 指向构建目标   |
| `$<`     | 指向第一个依赖 |
| `$^`     | 指向所有依赖   |

> 示例

- `Makefile`

```makefile
A = a.txt
OBJ = a.txt b.txt c.txt

init: $(A)
	cat $^

clean:
	rm $(OBJ)

a.txt: b.txt c.txt
	echo "target: $@\nfirst dependency: $<\nall dependencies: $^" > $@
b.txt:
	echo "target: $@" > $@
c.txt:
	echo "target: $@" > $@
```

执行结果：

```bash
$ make
echo "target: b.txt" > b.txt
echo "target: c.txt" > c.txt
echo "target: a.txt\nfirst dependency: b.txt\nall dependencies: b.txt c.txt" > a.txt
cat a.txt
target: a.txt
first dependency: b.txt
all dependencies: b.txt c.txt
$ make clean
rm a.txt b.txt c.txt
```

### 其他

makefile 还支持函数定义，也存在内置函数等，还有一些其他比较少用的特性，由于篇幅原因这里就不展开说明

# 结语

make 实在是一个方便简单的工具，相当于一个面向项目构建的 shell 脚本一般，透过 sh 脚本和 make、Makefile 的使用可以实现简单的自动化编译、构建、打包的操作，供大家参考。
