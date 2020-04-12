# Bash guide

1. [Advanced Bash-Scripting Guide](http://www.tldp.org/LDP/abs/html/)  
2. [ubuntu manuals for bash](http://manpages.ubuntu.com/manpages/eoan/en/man1/bash.1.html)

# Bash notes

## Array

## Meta
### 1. bash中 <<< 和 <<的区别是什么

<< 被称为 here-document 结构。 你可以让程序知道结束文本，并且每当看到该分隔符时，程序就会读取作为输入的任务。

```bash
strace -e open,dup2,pipe,write -f bash -c 'cat <<EOF
hello world
EOF'
```

<<< 被称为 here-string。 你不用在文本中键入文本，而是给程序预先设置一个文本字符串。 例如对于像 bc 这样的程序，我们可以为这个特定的例子做输出，而不需要交互地运行 bc
```bash
bc <<< "1+2"
```
