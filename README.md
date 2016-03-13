# bash-handbook-zh-CN

谨以此文档献给那些想学习Bash又无需钻研太深的人。

> **Tip**: 不妨尝试 [**learnyoubash**](https://git.io/learnyoubash) — 一个基于本文档的交互式学习平台！

# TODO Node

# TODO 目录

# 前言

如果你是一个程序员，时间的价值想必心中有数。持续优化工作流是你最重要的工作之一。

在通往高效和高生产力的路上，我们经常不得不做一些一遍又一遍重复的劳动，比如：

* 对屏幕截图，并把截图上传到服务器上
* 处理各种各种的文本
* 在不同格式之间转换文件
* 格式化一个程序的输出

就让**Bash**来拯救我们吧。

Bash是一个Unix Shell，作为[Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell)的free software替代品，由[Brian Fox][]为GNU项目编写。它发布于1989年，在很长一段时间，Linux系统和OS X系统都把Bash作为默认的shell。

[Brian Fox]: https://en.wikipedia.org/wiki/Brian_Fox_(computer_programmer)
<!-- 一些MD处理器不能很好的处理URL里面的 '()'，因此这个链接采用这种格式 -->

那么，我们学习这个有着30多年历史的东西意义何在呢？答案很简单：这个所谓的 _东西_，是当今最强大、可移植性最好的，为所有基于Unix的系统编写高效率脚本的工具之一。这就是你需要学习bash的原因。句号。

在本文中，我会用例子来描述在bash中最重要的思想。希望这篇概略性的文章对你有帮助。

# Shells与模式

bash shell有交互和非交互两种模式。

## 交互模式

Ubuntu用户都知道，在Ubuntu中有7个虚拟终端。
桌面环境接管了第7个虚拟终端，于是按下`Ctrl-Alt-F7`，便可以去到一个操作友好的图形用户界面。

即便如此，还是可以通过`Ctrl-Alt-F1`，来打开shell。打开后，熟悉的图形用户界面消失了，一个虚拟终端展现出来。

看到形如下面的东西，说明shell处于交互模式下：

    user@host:~$

接着，便可以输入一系列Unix命令，比如`ls`，`grep`，`cd`，`mkdir`，`rm`，来看它们的执行结果。

之所以把这种模式叫做交互式，是因为shell直接与用户进行交互。

直接使用虚拟终端其实并不是很方便。设想一下，当你想编辑一个文档，与此同时又想执行另一个命令，这种情况下，下面的虚拟终端模拟器可能更适合(**TODO**)：

- [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal)
- [Terminator](https://en.wikipedia.org/wiki/Terminator_(terminal_emulator))
- [iTerm2](https://en.wikipedia.org/wiki/ITerm2)
- [ConEmu](https://en.wikipedia.org/wiki/ConEmu)

## 非交互模式

在非交互模式下，shell从文件或者管道中读取命令并执行。当shell解释器执行完文件中的最后一个命令，进程终止，回到父进程(**TODO**)。

可以使用下面的命令让shell以非交互模式运行：

    sh /path/to/script.sh
    bash /path/to/script.sh

上面的例子中，`script.sh`是一个包含shell解释器可以识别并执行的命令的普通文本文件，`sh`和`bash`是shell解释器程序。你可以使用任何喜欢的编辑器创建`script.sh`（vim，nano，Sublime Text, Atom等等）。

除此之外，你还可以通过`chmod`命令给文件添加可执行的权限，来直接执行脚本文件：

    chmod +x /path/to/script.sh

这种方式要求脚本文件的第一行必须指明运行该脚本的程序，比如：

```bash
#!/bin/bash
echo "Hello, world!"
```
如果你更喜欢用`sh`，把`#!/bin/bash`改成`#!/bin/sh`即可。`#!`被称作[shebang](https://zh.wikipedia.org/wiki/Shebang)。之后，就能这样来运行脚本了：

    /path/to/script.sh

上面的例子中，我们使用了一个很有用的命令`echo`来输出字符串到屏幕上。

我们还可以这样来使用shebang：

```bash
#!/usr/bin/env bash
echo "Hello, world!"
```

这样做的好处是，系统会自动在`PATH`环境变量中查找你指定的程序（本例中的`bash`）。相比第一种写法，你应该尽量用这种写法，因为程序的路径是不确定的。这样写还有一个好处，操作系统的`PATH`变量有可能被配置为指向程序的另一个版本。比如，安装完新版本的`bash`，我们可能将其路径添加到`PATH`中，来“隐藏”老版本。如果直接用`#!/bin/bash`，那么系统会选择老版本的`bash`来执行脚本，如果用`#!/usr/bin/env bash`，则会使用新版本。


## 返回值

每个命令都有一个**返回值**（**返回状态**或者**退出状态**）。命令执行成功的返回值总是`0`（零值），执行失败的命令，返回一个非0值（错误码）。错误码必须是一个1到255之间的整数。

在编写脚本时，另一个很有用的命令是`exit`。这个命令被用来终止当前的执行，并把返回值交给shell。当`exit`不带任何参数时，它会终止当前脚本的执行并返回在它之前最后一个执行的命令的返回值。

一个程序运行结束后，shell将其**返回值**赋值给`$?`环境变量。因此`$?`变量通常被用来检测一个脚本执行成功与否。

与使用`exit`来结束一个脚本的执行类似，我们可以使用`return`命令来结束一个函数的执行并将**返回值**返回给调用者。当然，也可以在函数内部用`exit`，这 _不但_ 会中止函数的继续执行，_而且_ 会终止整个程序的执行。

# 注释

脚本中可以包含 _注释_。注释是特殊的语句，会被`shell`解释器忽略。它们以`#`开头，到行尾结束。

一个例子：

```bash
#!/bin/bash
# This script will print your username.
whoami
```

> **Tip**: 用注释来说明你的脚本是干什么的，以及 _为什么_ 这样写。

# 变量

跟许多程序设计语言一样，你可以在bash中创建变量。

Bash中没有数据类型。变量只能包含数字或者由一个或多个字符组成的字符串。你可以创建三种变量：局部变量，环境变量以及作为 _位置参数_ 的变量。

## 局部变量

**局部变量** 是仅在某个脚本内部有效的变量。它们不能被其他的程序和脚本访问。

局部变量可以用`=`声明（作为一种约定，变量名、`=`、变量的值之间 **不应该** 有空格），其值可以用`$`访问到。举个例子：

```bash
username="denysdovhan"  # 声明变量
echo $username          # 输出变量的值
unset username          # 删除变量
```

我们可以用`local`关键字声明属于某个函数的局部变量。这样声明的变量会在函数结束时消失。

```bash
local local_var="I'm a local value"
```

## 环境变量

**环境变量** 是对当前shell会话内所有的程序或脚本都可见的变量。创建它们跟创建局部变量类似，但使用的是`export`关键字。

```bash
export GLOBAL_VAR="I'm a global variable"
```

bash中有 _非常多_ 的环境变量。你会非常频繁地遇到它们，这里有一张速查表，记录了在实践中最常见的环境变量。

| Variable     | Description                                                   |
| :----------- | :------------------------------------------------------------ |
| `$HOME`      | 当前用户的用户目录                                              |
| `$PATH`      | 用分号分隔的目录列表，shell会到这些目录中查找命令                 |
| `$PWD`       | 当前工作目录                                                   |
| `$RANDOM`    | 0到32767之间的整数                                             |
| `$UID`       | 数值类型，当前用户的用户ID                                      |
| `$PS1`       | 主要系统输入提示符                                              |
| `$PS2`       | 次要系统输入提示符                                              |

[这里](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html#sect_03_02_04)有一张更全面的Bash环境变量列表。

## 位置参数

**位置参数** 是在调用一个函数并传给它参数时创建的变量。下表列出了在函数中，位置参数变量和一些其它的特殊变量以及它们的意义。

| Parameter      | Description                                                 |
| :------------- | :---------------------------------------------------------- |
| `$0`           | 脚本名称                                                     |
| `$1 … $9`      | 第1个到第9个参数列表                                          |
| `${10} … ${N}` | 第10个到N个参数列表                                           |
| `$*` or `$@`   | 除了`$0`外的所有位置参数                                      |
| `$#`           | 不包括`$0`在内的位置参数的个数                                 |
| `$FUNCNAME`    | 函数名称（仅在函数一个函数内部有值）                            |

在下面的例子中，位置参数为：`$0='./script.sh'`，`$1='foo'`，`$2='bar'`：

    ./script.sh foo bar

变量可以有 _默认_ 值。我们可以用如下语法来指定默认值：

```bash
 # 如果变量为空，赋给他们默认值
: ${VAR:='default'}
: ${$1:='first'}
# 或者
FOO=${FOO:-'default'}
```

# Shell 扩展

_扩展_ 发生在一行命令被分成一个个的 _记号（tokens）_ 之后。换言之，扩展是一种执行数学运算的机制，还可以用来保存命令的执行结果，等等。

感兴趣的话可以阅读[关于shell扩展的更多细节](https://www.gnu.org/software/bash/manual/bash.html#Shell-Expansions)。

## 大括号扩展

大括号扩展让生成任意的字符串成为可能。它跟 _文件名扩展_ 很类似，举个例子：

```bash
echo beg{i,a,u}n # begin began begun
```

大括号扩展还可以用来创建一个可被循环迭代的区间。

```bash
echo {0..5} # 0 1 2 3 4 5
echo {00..8..2} # 00 02 04 06 08
```
