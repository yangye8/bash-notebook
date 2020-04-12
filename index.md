# Bash notebook
  This records anything about bash language, tricky, notes when I learn and work with BASH.

Table of Contents
=================

   * [Bash notebook](#bash-notebook)
      * [Bash guide](#bash-guide)
      * [Bash notes](#bash-notes)
         * [Array](#array)
         * [Meta](#meta)
            * [1. bash中 &lt;&lt;&lt; 和 &lt;&lt;的区别是什么](#1-bash中--和-的区别是什么)
            * [2. bash中的 &lt;(some command) 进程替换](#2-bash中的-some-command-进程替换)
            * [3. Process substitution and pipe](#3-process-substitution-and-pipe)


## Bash guide

1. [Advanced Bash-Scripting Guide](http://www.tldp.org/LDP/abs/html/)  
2. [ubuntu manuals for bash](http://manpages.ubuntu.com/manpages/eoan/en/man1/bash.1.html)  
3. [markdown](https://github.com/google/styleguide/blob/gh-pages/docguide/style.md)
 
------

## Bash notes

### * Array

### * Meta
#### 1. bash中 <<< 和 <<的区别是什么
[HereDocument](http://mywiki.wooledge.org/HereDocument?action=show&redirect=HereString)

<< 被称为 `here-document` 结构。 你可以让程序知道结束文本，并且每当看到该分隔符时，程序就会读取作为输入的任务。

```shell
strace -e open,dup2,pipe,write -f bash -c 'cat <<EOF
hello world
EOF'
```

<<< 被称为 `here-string`。 你不用在文本中键入文本，而是给程序预先设置一个文本字符串。 例如对于像 bc 这样的程序，我们可以为这个特定的例子做输出，而不需要交互地运行 bc
```shell
bc <<< "1+2"
```
#### 2. bash中的 <(some command) 进程替换
[ProcessSubstitution](http://mywiki.wooledge.org/ProcessSubstitution) 
>Process substitution comes in two forms: <(some command) and >(some command). Each form either causes a FIFO to be created under /tmp or /var/tmp, or uses a named file descriptor (/dev/fd/*), depending on the operating system. The substitution syntax is replaced by the name of the FIFO or FD, and the command inside it is run in the background. The substitution is performed at the same time as parameter expansion and command substitution.  
One of the most common uses of this feature is to avoid the creation of temporary files, e.g. when using diff(1):
```shell
diff <(sort list1) <(sort list2)
#This is (roughly) equivalent to:

mkfifo /var/tmp/fifo1
mkfifo /var/tmp/fifo2
sort list1 >/var/tmp/fifo1 &
sort list2 >/var/tmp/fifo2 &
diff /var/tmp/fifo1 /var/tmp/fifo2
rm /var/tmp/fifo1 /var/tmp/fifo2
```
```shell
sh <(curl https://j.mp/spf13-vim3 -L)
```
the command actually receives **filename arguments**.  
Another common use is avoiding the loss of variables inside a loop that is part of a pipeline. 

```shell
i=0
while read line; do
((i++))
...
done < <(sort list1)
```
#### 3. Process substitution and pipe
Pipes and input redirects shove content onto the **STDIN stream**.  
Process substitution runs the commands, saves their output to a special temporary file and then passes that file name in place of the command. Whatever command you are using treats it as a **file name**. Note that the file created is not a regular file but a named pipe that gets removed automatically once it is no longer needed.

### 4. $0, $#, $*, $@, $?, $$
| name  | mean |
|-------|:---:|
|$0 |	当前脚本的文件名|
|$n |	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。|
|$# |	传递给脚本或函数的参数个数。|
|$* |	传递给脚本或函数的所有参数。|
|$@ |	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。|
|$? | 上个命令的退出状态，或函数的返回值。|
|$$ | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。|  

>当被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1\n" "$2\n" … "$n\n" 的形式输出所有参数

### 5. ()、(())、[]、[[]]、{}
（）
* 命令组。括号中的命令将会新开一个子shell顺序执行，所以括号中的变量不能够被脚本余下的部分使用。括号中多个命令之间用分号隔开，最后一个命令可以没有分号，各命令和括号之间不必有空格  
* 用于初始化数组。如：array=(a b c d)

(())
* 整数扩展。这种扩展计算是整数型的计算，不支持浮点型。((exp))结构扩展并计算一个算术表达式的值，如果表达式的结果为0，那么返回的退出状态码为1，或者 是"假"，而一个非零值的表达式所返回的退出状态码将为0，或者是"true"。若是逻辑判断，表达式exp为真则为1,假则为0。
* 只要括号中的运算符、表达式符合C语言运算规则，都可用在$((exp))中，甚至是三目运算符。作不同进位(如二进制、八进制、十六进制)运算时，输出结果全都自动转化成了十进制。如：echo $((16#5f)) 结果为95 (16进位转十进制)
* 单纯用 (( )) 也可重定义变量值，比如 a=5; ((a++)) 可将 $a 重定义为6
* 常用于算术运算比较，双括号中的变量可以不使用$符号前缀。括号内支持多个表达式用逗号分开。 只要括号中的表达式符合C语言运算规则,比如可以直接使用for((i=0;i<5;i++)), 如果不使用双括号, 则为for i in `seq 0 4`或者for i in {0..4}。再如可以直接使用if (($i<5)), 如果不使用双括号, 则为if [ $i -lt 5 ]。
