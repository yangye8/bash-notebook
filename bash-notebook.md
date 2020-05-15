# BASH NOTEBOOK

  This records anything about bash language, tricky, notes when I learn and work with BASH.

[TOC]

## Bash guide

[Advanced Bash-Scripting Guide](http://www.tldp.org/LDP/abs/html/)  
[ubuntu manuals for bash](http://manpages.ubuntu.com/manpages/eoan/en/man1/bash.1.html)  
[The bash shell](https://www.computerhope.com/unix/ubash.htm)
[bash-hackers](https://wiki.bash-hackers.org/)

[markdown](https://github.com/google/styleguide/blob/gh-pages/docguide/style.md)  
 
------

## Bash notes
<!-- CHAPTER START -->
### * Array

<!-- CHAPTER END -->

<!-- CHAPTER START -->
### * Trap

The trap command with no arguments shall write to standard output

```shell
    trap -- <action>, <condition>
```

If ARG is absent (and a single SIGNAL_SPEC is supplied) or `-`, each specified signal is *reset* to its original value.

```shell
    trap - <action>, <condition>
```
<!-- CHAPTER END -->

<!-- CHAPTER START -->
### * QA

#### 1. bash 中 <<< 和 <<的区别是什么

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

#### 2. bash 中的 <(some command) 进程替换

[ProcessSubstitution](http://mywiki.wooledge.org/ProcessSubstitution) 
>Process substitution comes in two forms: <(some command) and >(some command). Each form either causes a FIFO to be created under /tmp or /var/tmp, or uses a named file descriptor (/dev/fd/*), depending on the operating system. The substitution syntax is replaced by the name of the FIFO or FD, and the command inside it is run in the background. The substitution is performed at the same time as parameter expansion and command substitution.  
One of the most common uses of this feature is to avoid the creation of temporary files, e.g. when using diff(1):

```shell
diff <(sort list1) <(sort list2)
#THIS IS (ROUGHLY) EQUIVALENT TO:

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

* Pipes and input redirects shove content onto the **STDIN stream**.  
* Process substitution runs the commands, saves their output to a special temporary file and then passes that file name in place of the command. Whatever command you are using treats it as a **file name**. Note that the file created is not a regular file but a named pipe that gets removed automatically once it is no longer needed.

#### 4. $0, $#, $*, $@, $?, $$

| name  | mean |
|-------|:---:|
|$0 |    当前脚本的文件名|
|$n |    传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。|
|$# |    传递给脚本或函数的参数个数。|
|$* |    传递给脚本或函数的所有参数。|
|$@ |    传递给脚本或函数的所有参数。被双引号 (" ") 包含时，与 $* 稍有不同，下面将会讲到。|
|$? | 上个命令的退出状态，或函数的返回值。|
|$$ | 当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID。|  

>当被双引号 (" ") 包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1\n" "$2\n" … "$n\n" 的形式输出所有参数

#### 5. ()、(())、[]、[[]]、{}

()

* 命令组。括号中的命令将会新开一个子 shell 顺序执行   
* 用于初始化数组。如：array=(a b c d)

((  ))

* ((exp)) 结构扩展并计算一个算术表达式的值  
* 常用于算术运算比较，双括号中的变量可以不使用$符号前缀。括号内支持多个表达式用逗号分开。 for((i=0;i<5;i++))  

[ ]

* [和 test 等同。如果不用绝对路径，通常用 bash 自带的 [命令。if/test 结构中 [是调用 test 的命令标识，] 是关闭条件判断
* 字符范围。用作正则表达式的一部分，描述一个匹配的字符范围
* 在一个 array 结构的上下文中，中括号用来引用数组中每个元素的编号

[[]]

* [[是 bash 程序语言的关键字。并不是一个命令，[[ ]] 结构比 [ ] 结构更加通用。在 [[和]] 之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
* 支持字符串的模式匹配，使用=~操作符时甚至支持 shell 的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如 [[ hello == hell? ]]，结果为真。[[ ]] 中匹配字符串或通配符，不需要引号。
* 使用 [[ ... ]] 条件判断结构，而不是 [ ... ]，能够防止脚本中的许多逻辑错误。比如，&&、||、<和> 操作符能够正常存在于 [[ ]] 条件判断结构中，但是如果出现在 [ ] 结构中的话，会报错。比如可以直接使用 if [[ $a != 1 && $a != 2 ]], 如果不适用双括号，则为 if [ $a -ne 1] && [ $a != 2 ] 或者 if [ $a -ne 1 -a $a != 2 ]。
* bash 把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码

>>>数字运算： -eq -ne -lt -le -gt -ge，[[ ]] 同 [ ] 一致  
>>>文件运算： -r、-l、-w、-x、-f、-d、-s、-nt、-ot，[[ ]] 同 [ ] 一致  
>>>字符串 ： > < =（同==) != -n -z，不可使用"<="和">="，[[ ]] 同 [ ] 一致，但在 [] 中，>和<必须使用、进行转义，即、>和、<  
>>>逻辑运算： [] 为 -a -o ! [[ ]] 为&& || !  
>>>数学运算： [] 不可以使用 [[ ]] 可以使用+ - */ %  
  
{ }

* 大括号拓展。（通配 (globbing)) 将对大括号中的文件名做扩展。在大括号中，不允许有空白  
* 代码块，这个结构事实上创建了一个匿名函数 。与小括号中的命令不同，大括号内的命令不会新开一个子 shell 运行，即脚本余下部分仍可使用括号内变量。括号内的命令间用分号隔开，最后一个也必须有分号。{}的第一个命令和左括号之间必须要有一个空格 { cmd1;cmd2;cmd3;}

#### 6. difference in ">" and ">&"
`>`  means redirect output to a file.

`>&` means redirect output to another file descriptor

shell中描述符一共有12个

0  代表标准输入
1  代表标准输出
2  错误输出
其他 3-9 都是空白描述符

#### 7. 
<!-- CHAPTER END -->
