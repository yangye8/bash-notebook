<!-- TOC -->

- [高级bash脚本](#1-高级bash脚本)
    - [第27章 数组](#11-第27章数组)
        - [简单的数组应用](#111-简单的数组应用)
        - [例子27-2. 格式化输出一首诗](#112-例子27-2.格式化输出一首诗)
        - [例子27-3. 数组操作](#113-例子27-3.数组操作)
        - [示例27-4. 数组字里的符串操作](#114-示例27-4.数组字里的符串操作)
        - [示例27-5. 把脚本的内容加载到数组](#115-示例27-5.把脚本的内容加载到数组)
        - [示例27-6 数字的一些特殊属性](#116-示例27-6数字的一些特殊属性)
        - [示例27-7 空数组和空元素](#117-示例27-7空数组和空元素)
        - [示例27-8 初始化数组](#118-示例27-8初始化数组)

<!-- /TOC -->



<a id="toc_anchor" name="#1-高级bash脚本"></a>

# 1. 高级bash脚本

<a id="toc_anchor" name="#11-第27章数组"></a>

## 1.1. 第27章 数组



新版的bash支持一维数组，数组可以向 `variable[xx]` 这样被初始化，另外，脚本也可以显式声明一个数组 `declare -a variable` 

获取数组的值用 `${element[xx]}` 

<a id="toc_anchor" name="#111-简单的数组应用"></a>

### 1.1.1. 简单的数组应用

``` shell
#!/bin/bash

area[11]=23
area[13]=37
area[51]=UFOs

#  Array members need not be consecutive or contiguous.

#  Some members of the array can be left uninitialized.
#  Gaps in the array are okay.
#  In fact, arrays with sparse data ("sparse arrays")
#+ are useful in spreadsheet-processing software.

echo -n "area[11] = "
echo ${area[11]}    #  {curly brackets} needed.

echo -n "area[13] = "
echo ${area[13]}

echo "Contents of area[51] are ${area[51]}."

# Contents of uninitialized array variable print blank (null variable).
echo -n "area[43] = "
echo ${area[43]}
echo "(area[43] unassigned)"

echo

# Sum of two array variables assigned to third
area[5]= `expr ${area[11]} + ${area[13]}` 
echo "area[5] = area[11] + area[13]"
echo -n "area[5] = "
echo ${area[5]}

area[6]= `expr ${area[11]} + ${area[51]}` 
echo "area[6] = area[11] + area[51]"
echo -n "area[6] = "
echo ${area[6]}
# This fails because adding an integer to a string is not permitted.

echo; echo; echo

# -----------------------------------------------------------------
# Another array, "area2".
# Another way of assigning array variables...
# array_name=( XXX YYY ZZZ ... )

area2=( zero one two three four )

echo -n "area2[0] = "
echo ${area2[0]}
# Aha, zero-based indexing (first element of array is [0], not [1]).

echo -n "area2[1] = "
echo ${area2[1]}    # [1] is second element of array.
# -----------------------------------------------------------------

echo; echo; echo

# -----------------------------------------------
# Yet another array, "area3".
# Yet another way of assigning array variables...
# array_name=([xx]=XXX [yy]=YYY ...)

area3=([17]=seventeen [24]=twenty-four)

echo -n "area3[17] = "
echo ${area3[17]}

echo -n "area3[24] = "
echo ${area3[24]}
# -----------------------------------------------

exit 0
```

正如我们所看到的，初始化数组的简便方式是
`array=( elemetn1 element2 ... elementN )` 

``` shell
base64_charset=( {A..Z} {a..z} {0..9} + / = )
               #  Using extended brace expansion
               #+ to initialize the elements of the array.
               #  Excerpted from vladz's "base64.sh" script
               #+ in the "Contributed Scripts" appendix.
```

bash允许对字符串变量进行数组操作, 一个字符串是一个数组元素

``` shell
string=abcABC123ABCabc
echo ${string[@]}               # abcABC123ABCabc
echo ${string[*]}               # abcABC123ABCabc 
echo ${string[0]}               # abcABC123ABCabc
echo ${string[1]}               # No output!
                                # Why?
echo ${#string[@]}              # 1
                                # One element in the array.
                                # The string itself.
```

<a id="toc_anchor" name="#112-例子27-2.格式化输出一首诗"></a>

### 1.1.2. 例子27-2. 格式化输出一首诗

``` shell
#!/bin/bash
# poem.sh: Pretty-prints one of the ABS Guide author's favorite poems.

# Lines of the poem (single stanza).
Line[1]="I do not know which to prefer,"
Line[2]="The beauty of inflections"
Line[3]="Or the beauty of innuendoes,"
Line[4]="The blackbird whistling"
Line[5]="Or just after."
# Note that quoting permits embedding whitespace.

# Attribution.
Attrib[1]=" Wallace Stevens"
Attrib[2]="\"Thirteen Ways of Looking at a Blackbird\""
# This poem is in the Public Domain (copyright expired).

echo

tput bold   # Bold print.

for index in 1 2 3 4 5    # Five lines.
do
  printf "     %s\n" "${Line[index]}"
done

for index in 1 2          # Two attribution lines.
do
  printf "          %s\n" "${Attrib[index]}"
done

tput sgr0   # Reset terminal.
            # See 'tput' docs.

echo

exit 0
```

<a id="toc_anchor" name="#113-例子27-3.数组操作"></a>

### 1.1.3. 例子27-3. 数组操作

``` shell
#!/bin/bash
# array-ops.sh: More fun with arrays.

array=( zero one two three four five )
# Element 0   1   2    3     4    5

echo ${array[0]}       #  zero
echo ${array:0}        #  zero
                       #  Parameter expansion of first element,
                       #+ starting at position # 0 (1st character).
echo ${array:1}        #  ero
                       #  Parameter expansion of first element,
                       #+ starting at position # 1 (2nd character).

echo "--------------"

echo ${#array[0]}      #  4
                       #  Length of first element of array.
echo ${#array}         #  4
                       #  Length of first element of array.
                       #  (Alternate notation)

echo ${#array[1]}      #  3
                       #  Length of second element of array.
                       #  Arrays in Bash have zero-based indexing.

echo ${#array[*]}      #  6
                       #  Number of elements in array.
echo ${#array[@]}      #  6
                       #  Number of elements in array.

echo "--------------"

array2=( [0]="first element" [1]="second element" [3]="fourth element" )
#            ^     ^       ^     ^      ^       ^     ^      ^       ^
# Quoting permits embedding whitespace within individual array elements.

echo ${array2[0]}      # first element
echo ${array2[1]}      # second element
echo ${array2[2]}      #
                       # Skipped in initialization, and therefore null.
echo ${array2[3]}      # fourth element
echo ${#array2[0]}     # 13    (length of first element)
echo ${#array2[*]}     # 3     (number of elements in array)

exit
```

<a id="toc_anchor" name="#114-示例27-4.数组字里的符串操作"></a>

### 1.1.4. 示例27-4. 数组字里的符串操作

``` shell
#!/bin/bash
# array-strops.sh: String operations on arrays.

# Script by Michael Zick.
# Used in ABS Guide with permission.
# Fixups: 05 May 08, 04 Aug 08.

#  In general, any string operation using the ${name ... } notation
#+ can be applied to all string elements in an array,
#+ with the ${name[@] ... } or ${name[*] ...} notation.

arrayZ=( one two three four five five )

echo

# Trailing Substring Extraction
echo ${arrayZ[@]:0}     # one two three four five five
#                ^        All elements.

echo ${arrayZ[@]:1}     # two three four five five
#                ^        All elements following element[0].

echo ${arrayZ[@]:1:2}   # two three
#                  ^      Only the two elements after element[0].

echo "---------"

# Substring Removal

# Removes shortest match from front of string(s).

echo ${arrayZ[@]#f*r}   # one two three five five
#               ^       # Applied to all elements of the array.
                        # Matches "four" and removes it.

# Longest match from front of string(s)
echo ${arrayZ[@]##t*e}  # one two four five five
#               ^^      # Applied to all elements of the array.
                        # Matches "three" and removes it.

# Shortest match from back of string(s)
echo ${arrayZ[@]%h*e}   # one two t four five five
#               ^       # Applied to all elements of the array.
                        # Matches "hree" and removes it.

# Longest match from back of string(s)
echo ${arrayZ[@]%%t*e}  # one two four five five
#               ^^      # Applied to all elements of the array.
                        # Matches "three" and removes it.

echo "----------------------"

# Substring Replacement

# Replace first occurrence of substring with replacement.
echo ${arrayZ[@]/fiv/XYZ}   # one two three four XYZe XYZe
#               ^           # Applied to all elements of the array.

# Replace all occurrences of substring.
echo ${arrayZ[@]//iv/YY}    # one two three four fYYe fYYe
                            # Applied to all elements of the array.

# Delete all occurrences of substring.
# Not specifing a replacement defaults to 'delete' ...
echo ${arrayZ[@]//fi/}      # one two three four ve ve
#               ^^          # Applied to all elements of the array.

# Replace front-end occurrences of substring.
echo ${arrayZ[@]/#fi/XY}    # one two three four XYve XYve
#                ^          # Applied to all elements of the array.

# Replace back-end occurrences of substring.
echo ${arrayZ[@]/%ve/ZZ}    # one two three four fiZZ fiZZ
#                ^          # Applied to all elements of the array.

echo ${arrayZ[@]/%o/XX}     # one twXX three four five five
#                ^          # Why?

echo "-----------------------------"

replacement() {
    echo -n "!!!"
}

echo ${arrayZ[@]/%e/$(replacement)}
#                ^  ^^^^^^^^^^^^^^
# on!!! two thre!!! four fiv!!! fiv!!!
# The stdout of replacement() is the replacement string.
# Q.E.D: The replacement action is, in effect, an 'assignment.'

echo "------------------------------------"

#  Accessing the "for-each":
echo ${arrayZ[@]//*/$(replacement optional_arguments)}
#                ^^ ^^^^^^^^^^^^^
# !!! !!! !!! !!! !!! !!!

#  Now, if Bash would only pass the matched string
#+ to the function being called . . .

echo

exit 0

#  Before reaching for a Big Hammer -- Perl, Python, or all the rest --
#  recall:
#    $( ... ) is command substitution.
#    A function runs as a sub-process.
#    A function writes its output (if echo-ed) to stdout.
#    Assignment, in conjunction with "echo" and command substitution,
#+   can read a function's stdout.
#    The name[@] notation specifies (the equivalent of) a "for-each"
#+   operation.
#  Bash is more powerful than you think!
```

<a id="toc_anchor" name="#115-示例27-5.把脚本的内容加载到数组"></a>

### 1.1.5. 示例27-5. 把脚本的内容加载到数组

``` shell
#!/bin/bash
# script-array.sh: Loads this script into an array.
# Inspired by an e-mail from Chris Martin (thanks!).

script_contents=( $(cat "$0") )  #  Stores contents of this script ($0)
                                 #+ in an array.

for element in $(seq 0 $((${#script_contents[@]} - 1)))
  do                #  ${#script_contents[@]}
                    #+ gives number of elements in the array.
                    #
                    #  Question:
                    #  Why is  seq 0  necessary?
                    #  Try changing it to seq 1.
  echo -n "${script_contents[$element]}"
                    # List each field of this script on a single line.
# echo -n "${script_contents[element]}" also works because of ${ ... }.
  echo -n " -- "    # Use " -- " as a field separator.
done

echo

exit 0

# Exercise:
# --------
#  Modify this script so it lists itself
#+ in its original format,
#+ complete with whitespace, line breaks, etc.
```

`unset` 内建命令可以删除数组元素，也可以删除整个数组

<a id="toc_anchor" name="#116-示例27-6数字的一些特殊属性"></a>

### 1.1.6. 示例27-6 数字的一些特殊属性

``` shell
#!/bin/bash

declare -a colors
#  All subsequent commands in this script will treat
#+ the variable "colors" as an array.

echo "Enter your favorite colors (separated from each other by a space)."

read -a colors    # Enter at least 3 colors to demonstrate features below.
#  Special option to 'read' command,
#+ allowing assignment of elements in an array.

echo

element_count=${#colors[@]}
# Special syntax to extract number of elements in array.
#     element_count=${#colors[*]} works also.
#
#  The "@" variable allows word splitting within quotes
#+ (extracts variables separated by whitespace).
#
#  This corresponds to the behavior of "$@" and "$*"
#+ in positional parameters. 

index=0

while [ "$index" -lt "$element_count" ]
do    # List all the elements in the array.
  echo ${colors[$index]}
  #    ${colors[index]} also works because it's within ${ ... } brackets.
  let "index = $index + 1"
  # Or:
  #    ((index++))
done
# Each array element listed on a separate line.
# If this is not desired, use  echo -n "${colors[$index]} "
#
# Doing it with a "for" loop instead:
#   for i in "${colors[@]}"
#   do
#     echo "$i"
#   done
# (Thanks, S.C.)

echo

# Again, list all the elements in the array, but using a more elegant method.
  echo ${colors[@]}          # echo ${colors[*]} also works.

echo

# The "unset" command deletes elements of an array, or entire array.
unset colors[1]              # Remove 2nd element of array.
                             # Same effect as   colors[1]=
echo  ${colors[@]}           # List array again, missing 2nd element.

unset colors                 # Delete entire array.
                             #  unset colors[*] and
                             #+ unset colors[@] also work.
echo; echo -n "Colors gone."			   
echo ${colors[@]}            # List array again, now empty.

exit 0
```

从例子中看出，无论是 `${array_name[@]}` 还是 `${array_name[*]}` 都代表数组中的所有元素，类似的
获取数组中的元素总数用 `${#array_name[@]}` 或者 `${#array_name[*]}` 
`${#array_name}` 是数组第一个元素 `${array_name[0]}` 的长度（多少个字符）

<a id="toc_anchor" name="#117-示例27-7空数组和空元素"></a>

### 1.1.7. 示例27-7 空数组和空元素

``` shell
#!/bin/bash
# empty-array.sh

#  Thanks to Stephane Chazelas for the original example,
#+ and to Michael Zick and Omair Eshkenazi, for extending it.
#  And to Nathan Coulter for clarifications and corrections.

# An empty array is not the same as an array with empty elements.

  array0=( first second third )
  array1=( '' )   # "array1" consists of one empty element.
  array2=( )      # No elements . . . "array2" is empty.
  array3=(   )    # What about this array?

echo
ListArray()
{
echo
echo "Elements in array0:  ${array0[@]}"
echo "Elements in array1:  ${array1[@]}"
echo "Elements in array2:  ${array2[@]}"
echo "Elements in array3:  ${array3[@]}"
echo
echo "Length of first element in array0 = ${#array0}"
echo "Length of first element in array1 = ${#array1}"
echo "Length of first element in array2 = ${#array2}"
echo "Length of first element in array3 = ${#array3}"
echo
echo "Number of elements in array0 = ${#array0[*]}"  # 3
echo "Number of elements in array1 = ${#array1[*]}"  # 1  (Surprise!)
echo "Number of elements in array2 = ${#array2[*]}"  # 0
echo "Number of elements in array3 = ${#array3[*]}"  # 0
}

# ===================================================================

ListArray

# Try extending those arrays.

# Adding an element to an array.
array0=( "${array0[@]}" "new1" )
array1=( "${array1[@]}" "new1" )
array2=( "${array2[@]}" "new1" )
array3=( "${array3[@]}" "new1" )

ListArray

# or
array0[${#array0[*]}]="new2"
array1[${#array1[*]}]="new2"
array2[${#array2[*]}]="new2"
array3[${#array3[*]}]="new2"

ListArray

# When extended as above, arrays are 'stacks' ...
# Above is the 'push' ...
# The stack 'height' is:
height=${#array2[@]}
echo
echo "Stack height for array2 = $height"

# The 'pop' is:
unset array2[${#array2[@]}-1]   #  Arrays are zero-based,
height=${#array2[@]}            #+ which means first element has index 0.
echo
echo "POP"
echo "New stack height for array2 = $height"

ListArray

# List only 2nd and 3rd elements of array0.
from=1		    # Zero-based numbering.
to=2
array3=( ${array0[@]:1:2} )
echo
echo "Elements in array3:  ${array3[@]}"

# Works like a string (array of characters).
# Try some other "string" forms.

# Replacement:
array4=( ${array0[@]/second/2nd} )
echo
echo "Elements in array4:  ${array4[@]}"

# Replace all matching wildcarded string.
array5=( ${array0[@]//new?/old} )
echo
echo "Elements in array5:  ${array5[@]}"

# Just when you are getting the feel for this . . .
array6=( ${array0[@]#*new} )
echo # This one might surprise you.
echo "Elements in array6:  ${array6[@]}"

array7=( ${array0[@]#new1} )
echo # After array6 this should not be a surprise.
echo "Elements in array7:  ${array7[@]}"

# Which looks a lot like . . .
array8=( ${array0[@]/new1/} )
echo
echo "Elements in array8:  ${array8[@]}"

#  So what can one say about this?

#  The string operations are performed on
#+ each of the elements in var[@] in succession.
#  Therefore : Bash supports string vector operations.
#  If the result is a zero length string,
#+ that element disappears in the resulting assignment.
#  However, if the expansion is in quotes, the null elements remain.

#  Michael Zick:    Question, are those strings hard or soft quotes?
#  Nathan Coulter:  There is no such thing as "soft quotes."
#!    What's really happening is that
#!+   the pattern matching happens after
#!+   all the other expansions of [word]
#!+   in cases like ${parameter#word}.

zap='new*'
array9=( ${array0[@]/$zap/} )
echo
echo "Number of elements in array9:  ${#array9[@]}"
array9=( "${array0[@]/$zap/}" )
echo "Elements in array9:  ${array9[@]}"
# This time the null elements remain.
echo "Number of elements in array9:  ${#array9[@]}"

# Just when you thought you were still in Kansas . . .
array10=( ${array0[@]#$zap} )
echo
echo "Elements in array10:  ${array10[@]}"
# But, the asterisk in zap won't be interpreted if quoted.
array10=( ${array0[@]#"$zap"} )
echo
echo "Elements in array10:  ${array10[@]}"
# Well, maybe we _are_ still in Kansas . . .
# (Revisions to above code block by Nathan Coulter.)

#  Compare array7 with array10.
#  Compare array8 with array9.

#  Reiterating: No such thing as soft quotes!
#  Nathan Coulter explains:
#  Pattern matching of 'word' in ${parameter#word} is done after
#+ parameter expansion and *before* quote removal.
#  In the normal case, pattern matching is done *after* quote removal.
 
exit
```

`${array_name[@]}` 和 `${array_name[*]}` 的关系和 `$@` and `$*` 类似

``` shell
# Copying an array.
array2=( "${array1[@]}" )
# or
array2="${array1[@]}"
#
#  However, this fails with "sparse" arrays,
#+ arrays with holes (missing elements) in them,
#+ as Jochen DeSmet points out.
# ------------------------------------------
  array1[0]=0
# array1[1] not assigned
  array1[2]=2
  array2=( "${array1[@]}" )       # Copy it?

echo ${array2[0]}      # 0
echo ${array2[2]}      # (null), should be 2
# ------------------------------------------
# Adding an element to an array.
array=( "${array[@]}" "new element" )
# or
array[${#array[*]}]="new element"

# Thanks, S.C.
```

``` shell
#!/bin/bash

filename=sample_file

#            cat sample_file
#
#            1 a b c
#            2 d e fg

declare -a array1

array1=( `cat "$filename"` )                #  Loads contents
#         List file to stdout              #+ of $filename into array1.
#
#  array1=( `cat "$filename" | tr '\n' ' '` )
#                            change linefeeds in file to spaces. 
#  Not necessary because Bash does word splitting,
#+ changing linefeeds to spaces.

echo ${array1[@]}            # List the array.
#                              1 a b c 2 d e fg
#
#  Each whitespace-separated "word" in the file
#+ has been assigned to an element of the array.

element_count=${#array1[*]}
echo $element_count          # 8
```

<a id="toc_anchor" name="#118-示例27-8初始化数组"></a>

### 1.1.8. 示例27-8 初始化数组

``` shell
#! /bin/bash
# array-assign.bash

#  Array operations are Bash-specific,
#+ hence the ".bash" in the script name.

# Copyright (c) Michael S. Zick, 2003, All rights reserved.
# License: Unrestricted reuse in any form, for any purpose.
# Version: $ID$
#
# Clarification and additional comments by William Park.

#  Based on an example provided by Stephane Chazelas
#+ which appeared in an earlier version of the
#+ Advanced Bash Scripting Guide.

# Output format of the 'times' command:
# User CPU <space> System CPU
# User CPU of dead children <space> System CPU of dead children

#  Bash has two versions of assigning all elements of an array
#+ to a new array variable.
#  Both drop 'null reference' elements
#+ in Bash versions 2.04 and later.
#  An additional array assignment that maintains the relationship of
#+ [subscript]=value for arrays may be added to newer versions.

#  Constructs a large array using an internal command,
#+ but anything creating an array of several thousand elements
#+ will do just fine.

declare -a bigOne=( /dev/* )  # All the files in /dev . . .
echo
echo 'Conditions: Unquoted, default IFS, All-Elements-Of'
echo "Number of elements in array is ${#bigOne[@]}"

# set -vx

echo
echo '- - testing: =( ${array[@]} ) - -'
times
declare -a bigTwo=( ${bigOne[@]} )
# Note parens:    ^              ^
times

echo
echo '- - testing: =${array[@]} - -'
times
declare -a bigThree=${bigOne[@]}
# No parentheses this time.
times

#  Comparing the numbers shows that the second form, pointed out
#+ by Stephane Chazelas, is faster.
#
#  As William Park explains:
#+ The bigTwo array assigned element by element (because of parentheses),
#+ whereas bigThree assigned as a single string.
#  So, in essence, you have:
#                   bigTwo=( [0]="..." [1]="..." [2]="..." ... )
#                   bigThree=( [0]="... ... ..." )
#
#  Verify this by:  echo ${bigTwo[0]}
#                   echo ${bigThree[0]}

#  I will continue to use the first form in my example descriptions
#+ because I think it is a better illustration of what is happening.

#  The reusable portions of my examples will actual contain
#+ the second form where appropriate because of the speedup.

# MSZ: Sorry about that earlier oversight folks.

#  Note:
#  ----
#  The "declare -a" statements in lines 32 and 44
#+ are not strictly necessary, since it is implicit
#+ in the  Array=( ... )  assignment form.
#  However, eliminating these declarations slows down
#+ the execution of the following sections of the script.
#  Try it, and see.

exit 0
```
在数组声明中添加`declare -a`语句可以加快数组后续操作的执行速度。



### 示例27-9 copy和级联数组
```shell
#! /bin/bash
# CopyArray.sh
#
# This script written by Michael Zick.
# Used here with permission.

#  How-To "Pass by Name & Return by Name"
#+ or "Building your own assignment statement".


CpArray_Mac() {

# Assignment Command Statement Builder

    echo -n 'eval '
    echo -n "$2"                    # Destination name
    echo -n '=( ${'
    echo -n "$1"                    # Source name
    echo -n '[@]} )'

# That could all be a single command.
# Matter of style only.
}

declare -f CopyArray                # Function "Pointer"
CopyArray=CpArray_Mac               # Statement Builder

Hype()
{

# Hype the array named $1.
# (Splice it together with array containing "Really Rocks".)
# Return in array named $2.

    local -a TMP
    local -a hype=( Really Rocks )

    $($CopyArray $1 TMP)
    TMP=( ${TMP[@]} ${hype[@]} )
    $($CopyArray TMP $2)
}

declare -a before=( Advanced Bash Scripting )
declare -a after

echo "Array Before = ${before[@]}"

Hype before after

echo "Array After = ${after[@]}"

# Too much hype?

echo "What ${after[@]:3:2}?"

declare -a modest=( ${after[@]:2:1} ${after[@]:3:2} )
#                    ---- substring extraction ----

echo "Array Modest = ${modest[@]}"

# What happened to 'before' ?

echo "Array Before = ${before[@]}"

exit 0
```
