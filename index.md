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
当被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数
