https://wiki.bash-hackers.org/howto/getopts_tutorial

# 如何处理位置参数

## 位置参数

    | $0               | 第一个位置参数，相当于 `argv[0]` 
    | $FUNCNAME        | 函数名
    | $1 ... $9        | 参数表 从1到9
    | ${10} ... ${N}   | 超过9的参数表
    | $*               | 除了 `$0` 外的所有的位置参数 $1 $2 $3 ... ${N} 
    | $@               | 除了 `$0` 外的所有的位置参数 $1 $2 $3 ... ${N} 
    | "$*"             | 除了 `$0` 外的所有的位置参数 "$1IFS$2IFS$3IFS... ${N}" 
    | "$@"             | 除了 `$0` 外的所有的位置参数 "$1" "$2" "$3" ... "${N}"
    | $#               | 参数个数，不含 `$0` 

## 提取位置参数

``` she
${@:start:count}
${*:start:count}
"${@:start:count}"
"${*:start:count}"
```

如果start是负数，位置参数倒序

`echo "${@:-1}"` # 从最后一个数开始

## 设置位置参数

脚本或者函数内部可以用 `set` 修改位置参数的值

``` shell
set "This is" my new "set of" positional parameters

RESULTS IN
$1: This is
$2: my
$3: new
$4: set of
$5: positional
$6: parameters
```

``` shell
$ set -- "arg  1" "arg  2" "arg  3"

$ for word in $*; do echo "$word"; done
arg
1
arg
2
arg
3

$ for word in $@; do echo "$word"; done
arg
1
arg
2
arg
3

$ for word in "$*"; do echo "$word"; done
arg  1 arg  2 arg  3

$ for word in "$@"; do echo "$word"; done
arg  1
arg  2
arg  3
```


## $0

`$0` 用于表示执行脚本的名字
在一个函数内部， `$0` 还是表示脚本名，获取函数名用 `$FUNCNAME` 

## 移位

内建命令 `shift` 用于改变 位置参数的值
执行一次 `shift` 命令后

* `$1` 被丢弃
* `$2` 会变成 `$1` 
* `$3` 会变成 `$2` 
* ... 
* `$N` 会变成 `$N-1` 

shfit的默认移位是1, 也可以指定移位值

## 例子

### 固定个数

``` shell
#!/bin/bash
echo "Total number of arguments: $#"
echo "Argument 1: $1"
echo "Argument 2: $2"
echo "Argument 3: $3"
echo "Argument 4: $4"
echo "Argument 5: $5"
```

### 循环

#### 遍历参数

##### $#

每一次循环， `shift` 命令都会对参数表移位

``` shell
num=$#;
for ((i=1;i<=$num;i++))
do
    echo $i
    shift
done
```

`num` 用来记录最开始的参数个数，因为循环 `shift` 后，参数个数都会被改变

#### $@

另一种方法是没有给定字符串，每次用for循环遍历一个参数。for循环用所有位置参数作为选项参数 `$@` 

``` shell
for arg  # 等价于  for arg in "$@"
do 
    echo $arg
done
```

优点是 位置参数保存完好，因为没shift

#### $1

移位并检查 `$1` 还有没有

``` shell
while [[ "$1" ]]
do
    echo $1
    shift
done
```

缺点：如果$1里面有空字符作为参数

## 复杂应用

_________

# getopts 教程

## 描述

注意 getopts 既不能解析GNU-类型的长选项（--myoption）又不能解析XF86-类型那个的长选项（-myoption），因此，当你想解析命令行参数用getopts不专业，这时可以用getopt代替，这是shell内建的命令，好处如下：

* 不需要传位置参数
* 内建函数，getopts 可以设置shell变量用于解析

其它解析位置参数的方法，不用getopt和getopts，可以看这个 如何解析位置参数

## 术语

考虑下下面这条命令

``` shell
scrtip -x -f /etc/a.conf -r ./foo.txt ./bar.txt
```

这都是位置参数，但是可以分成几个逻辑组

* -x 是个选项，由-跟着一个字符组成
* -f 也是选项，但是它有选项相关的参数，选项参数不是必须的
* -r 依赖与配置，这个例子中它不带参数
* . /foo. txt 和. /bar. txt 是没有选项的参数

## 如何工作

通常，需要调getopts多次，每一次要用下个位置参数和可能的选项参数，getopts不会改变位置参数，如果你想移位，必须手动完成

``` shell
shift $((OPTIND-1))
```

当没有参数可解析的时候, getopts会返回FALSE退出，因此getopts很容易用在 while-loop 语句中

``` shell
while getopts ...; do
...
done
```

getopts将会解析参数项以及它们的位置参数，终止与第一个非选项参数(非-开头的字符串以及--)

## 常用的变量

变量     | 描述
OPTIND  | 下一个带解析参数的索引值，初始值是1, 每次getopts解析后要重置为1
OPTARG  | 记录getopts找到的所有参数选项

## 语法

getopts的基本语法

``` shell
getopts OPTSTRING VARNAME [ARGS...]
```

这里

* OPTSTRING       告诉getopts 有那些选项预期的选项和参数
* VARNAME         告诉getopts 那个shell变量用于选项解析
* ARGS            告诉getopts 解析ARGS而不是 位置参数

### 选项字串

选项字串告诉getopts有那些预期的选项以及参数

``` shell
getopts fAx VARNAME
```

这个选项字串告诉 *getopts* 找-f -A -x
如果选项带参数的话，需要在选项后跟上一个 `:` 告诉getopts，这是带参数的选项
如果选项字串的第一个字符是 `:` ，getopts会切换到静默模式

### 定制的的解析参数

`getops` 默认会解析当前shell和函数的位置参数，即解析 `"$@"` 
当然你可以设置自己的参数，通过ARGS变量

``` shell
while getopts :f:h opt "${MY[@]}"
do
done
```

如果不带ARGS参数，getopts默认执行 `"$@"` 

### 错误报告

`getops` 有两种模式

* 调试模式

无效操作                VARNAME是？并且 OPTARG 未设置
要求的参数没有找到      VARNAME是？并且OPTARG未设置，输出错误消息

* 静默模式

无效操作                VARNAME是？并且 OPTARG 设置成无效字符
要求的参数没有找到      VARNAME是：并且OPTARG包含选项字符

## 开始使用

### 第一个例子-不带参数

``` shell
while getopts ":a" opt;do
case $opt in
    a)
        echo "-a was triggered" >&2
        ;;
    \?)
        echo "Invalid option: -$OPTARG" >&2
        ;;
esac
done
```

几点注意

* 遇到无效选项， `getopts` 不会停止处理，只能在合适位置手动 `exit` 
* 多个相同的选项，需要人工检查处理

### 第二个例子-带参数

``` shell
while getopts ":a:" opt; do
case $opt in
    a)
        echo "-a was triggered, $OPTARG" >$2
        ;;
    \?)
        echo "Invalid option: -$OPTARG" >$2
        exit 1
        ;;
    :)
        echo "option -$OPTARG requires an arg" >$2
        exit 1
        ;;
    esac
done
```

go_test. sh -a

Option -a requires an argument 

go_test. sh -a /etc/passwd 

-a was triggered, Parameter: /etc/passwd
