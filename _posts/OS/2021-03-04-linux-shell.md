---
layout: post
title: "Linux Shell"
subtitle: "Shell学习笔记"
date: 2021-03-04
author: "Dictria"
header-img: "img/header.jpg"
tags: 
  - Linux
---

# 注意要点

# 变量

## 定义变量

```shell
variable=value
variable='value'
variable="value"
```

**赋值号=的周围不能有空格**  

## 使用变量

* 使用一个定义过的变量，只要在变量名前面加美元符号`$`即可
* 变量名外面的花括号`{ }`是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，例如：

```shell
skill="Java"
echo "I am good at ${skill}Script"
```

## 修改变量的值

已定义的变量，可以被重新赋值，如：

```shell
url="http://c.biancheng.net"
echo ${url}
url="http://c.biancheng.net/shell/"
echo ${url}
```

对变量赋值时不能在变量名前加`$`，只有在使用变量时才能加`$`  

## 单双引号的区别

变量的值可以由单引号`' '`包围，也可以由双引号`" "`包围

```shell
#!/bin/bash
url="http://c.biancheng.net"
website1='C语言中文网：${url}'
website2="C语言中文网：${url}"
echo $website1
echo $website2
```

运行结果：

```shell
C语言中文网：${url}
C语言中文网：http://c.biancheng.net
```

以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。  
以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。  

## 将命令的结果赋值给变量

将命令的执行结果赋值给变量，常见的有以下两种方式：  

```shell
variable=`command`
variable=$(command)
```

## 只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。  
下面的例子尝试更改只读变量，结果报错：

```shell
#!/bin/bash
myUrl="http://c.biancheng.net/shell/"
readonly myUrl
myUrl="http://c.biancheng.net/shell/"
```

## 删除变量

使用 unset 命令可以删除变量。语法：  
`unset variable_name`  
变量被删除后不能再次使用；unset 命令不能删除只读变量。  

## 变量作用域

Shell 变量的作用域可以分为三种：

* 只能在函数内部使用，这叫做局部变量（local variable）；
* 可以在当前 Shell 进程中使用，这叫做全局变量（global variable）；
* 可以在子进程中使用，这叫做环境变量（environment variable）。  
  **在 Shell 中定义的变量，默认就是全局变量。**

### 局部变量

在 Shell 函数中定义的变量默认也是全局变量，它和在函数外部定义变量拥有一样的效果。  

```shell
#!/bin/bash
#定义函数
function func(){
    a=99
}
#调用函数
func
#输出函数内部的变量
echo $a
```

输出结果：99  
&nbsp;
要想变量的作用域仅限于函数内部，可以在定义时加上local命令，此时该变量就成了局部变量。

```shell
#!/bin/bash
#定义函数
function func(){
    local a=99
}
#调用函数
func
#输出函数内部的变量
echo $a
```

输出结果为空

### 全局变量

**在当前 Shell 进程中使用**  

### 环境变量

全局变量只在当前 Shell 进程中有效，对其它 Shell 进程和子进程都无效。如果使用`export`命令将全局变量导出，那么它就在所有的子进程中也有效了，这称为“环境变量”。  
环境变量被创建时所处的 Shell 进程称为父进程，如果在父进程中再创建一个新的进程来执行 Shell 命令，那么这个新的进程被称作 Shell 子进程。当 Shell 子进程产生时，它会继承父进程的环境变量为自己所用  
**可以通过bash命令生成shell子进程**   
通过`exit`命令可以一层一层地退出 Shell。  

```shell
[c.biancheng.net]$ a=22       #定义一个全局变量
[c.biancheng.net]$ echo $a    #在当前Shell中输出a，成功
22
[c.biancheng.net]$ bash       #进入Shell子进程
[c.biancheng.net]$ echo $a    #在子进程中输出a，失败

[c.biancheng.net]$ exit       #退出Shell子进程，返回上一级Shell
exit
[c.biancheng.net]$ export a   #将a导出为环境变量
[c.biancheng.net]$ bash       #重新进入Shell子进程
[c.biancheng.net]$ echo $a    #在子进程中再次输出a，成功
22
[c.biancheng.net]$ exit       #退出Shell子进程
exit
[c.biancheng.net]$ exit       #退出父进程，结束整个Shell会话
```

`export a`这种形式是在定义变量 a 以后再将它导出为环境变量，如果想在定义的同时导出为环境变量，可以写作`export a=22`。  

## 将命令的输出结果赋给变量

Shell 中有两种方式可以完成命令替换，一种是反引号` `，一种是$()，使用方法如下：  

```shell
variable=`commands`
variable=$(commands)
```

**\$() 支持嵌套，反引号不行。**  
**$() 仅在 Bash Shell 中有效，而反引号可在多种 Shell 中使用。**  
**commands 可以只有一个命令，也可以有多个命令，多个命令之间以分号`;`分隔。**  
注意，如果被替换的命令的输出内容**包括多行（也即有换行符）**，或者含有多个连续的空白符，那么在输出变量时应该将变量用**双引号**包围，否则系统会使用默认的空白符来填充，这会导致换行无效，以及连续的空白符被压缩成一个。  

# 位置参数

运行 Shell 脚本文件时我们可以给它传递一些参数，这些参数在脚本文件内部可以使用`$n`的形式来接收，例如，`$1` 表示第一个参数，`$2` 表示第二个参数，依次类推。  

* 注意：如果参数个数太多，达到或者超过了 10 个，那么就得用`${n}`的形式来接收了，例如 `${10}`、`${23}`。`{ }`的作用是为了帮助解释器识别参数的边界，这跟使用变量时加`{ }`是一样的效果。

## 给脚本传递位置参数

```shell
#!/bin/bash
echo "Language: $1"
echo "URL: $2"
```

运行 test.sh，并附带参数：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ . ./test.sh Shell http://c.biancheng.net/shell/
Language: Shell
URL: http://c.biancheng.net/shell/
```

## 给函数传递位置参数

```shell
#!/bin/bash
#定义函数
function func(){
    echo "Language: $1"
    echo "URL: $2"
}
#调用函数
func C++ http://c.biancheng.net/cplus/
```

# 特殊变量

|                             变量                             |                             含义                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                              $0                              |                      当前脚本的文件名。                      |
| $n(n>=1)|传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 $1，第二个参数是 $2。 |                                                              |
|                              $#                              |                 传递给脚本或函数的参数个数。                 |
|                              $*                              |                 传递给脚本或函数的所有参数。                 |
|                             \$@                              | 传递给脚本或函数的所有参数。当被双引号`" "`包含时，`$@` 与 `$*` 稍有不同 |
|                              $?                              |              上个命令的退出状态，或函数的返回值              |
|                              $$                              | 当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID。 |
|                            &nbsp;                            |                                                              |

## $*和$@

$* 和 $@ 都表示传递给函数或脚本的所有参数，当它们被双引号" "包含时：

* `"$*"`会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。
* `"$@"`仍然将每个参数都看作一份数据，彼此之间是独立的。

```shell
#!/bin/bash
echo "print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done
echo "print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```

运行 test.sh，并附带参数：

```shell
[mozhiyan@localhost demo]$ . ./test.sh a b c d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d
```

## $?

### 获取上一命令的退出状态

```shell
#!/bin/bash
if [ "$1" == 100 ]
then
   exit 0  #参数正确，退出状态为0
else
   exit 1  #参数错误，退出状态1
fi
```

exit表示退出当前 Shell 进程，我们必须在新进程中运行 test.sh，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法取得它的退出状态了。  

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ bash ./test.sh 100  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
0
```

```shell
[mozhiyan@localhost demo]$ bash ./test.sh 89  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
1
```

### 获取函数的返回值

```shell
#!/bin/bash
#得到两个数相加的和
function add(){
    return `expr $1 + $2`
}
add 23 50  #调用函数
echo $?  #获取函数返回值
```

运行结果：73  

# 字符串

字符串可以由单引号' '包围，也可以由双引号" "包围，也可以不用引号。它们之间是有区别的：

* 由单引号`' '`包围的字符串：
  * 任何字符都会原样输出，在其中使用变量是无效的。
  * 字符串中不能出现单引号，即使对单引号进行转义也不行。
* 由双引号`" "`包围的字符串：
  * 如果其中包含了某个变量，那么该变量会被解析（得到该变量的值），而不是原样输出。
  * 字符串中可以出现双引号，只要它被转义了就行。
* 不被引号包围的字符串:
  * 不被引号包围的字符串中出现变量时也会被解析，这一点和双引号`" "`包围的字符串一样。
  * 字符串中不能出现空格，否则空格后边的字符串会作为其他变量或者命令解析。

## 获取字符串长度

`${#string_name}`  

## 字符串拼接

```shell
#!/bin/bash
name="Shell"
url="http://c.biancheng.net/shell/"
str1=$name$url  #中间不能有空格
str2="$name $url"  #如果被双引号包围，那么中间可以有空格
str3=$name": "$url  #中间可以出现别的字符串
str4="$name: $url"  #这样写也可以
str5="${name}Script: ${url}index.html"  #这个时候需要给变量名加上大括号
echo $str1
echo $str2
echo $str3
echo $str4
echo $str5
```

## 字符串截取

### 从指定位置开始截取

#### 从字符串左边开始截取

`${string: start :length}`  
string 是要截取的字符串，start 是起始位置（从左边开始，从 0 开始计数），length 是要截取的长度（省略的话表示直到字符串的末尾）。  

#### 从字符串右边开始截取

`${string: 0-start :length}`  

* 从左边开始计数时，起始数字是 0；从右边开始计数时，起始数字是 1。
* 不管从哪边开始计数，截取方向都是从左到右。

### 从指定字符（子字符串）开始截取

#### 使用 # 号截取右边字符

使用#号可以截取指定字符（或者子字符串）右边的所有字符，具体格式如下：  
`${string#*chars}`  
string 表示要截取的字符，chars 是指定的字符（或者子字符串），`*`是通配符的一种，表示任意长度的字符串。`*chars`连起来使用的意思是：忽略左边的所有字符，直到遇见 chars（chars 不会被截取）。    
如果不需要忽略 chars 左边的字符，那么也可以不写`*`  
以上写法遇到第一个匹配的字符（子字符串）就结束了  
如果希望直到最后一个指定字符（子字符串）再匹配结束，那么可以使用##，具体格式为：  
`${string##*chars}`  

```shell
#!/bin/bash
url="http://c.biancheng.net/index.html"
echo ${url#*/}    #结果为 /c.biancheng.net/index.html
echo ${url##*/}   #结果为 index.html
```

#### 使用%截取左边的字符

使用`%`号可以截取指定字符（或者子字符串）左边的所有字符，具体格式如下：  
`${string%chars*}`  
请注意*的位置，因为要截取 chars 左边的字符，而忽略 chars 右边的字符，所以*应该位于 chars 的右侧。其他方面%和#的用法相同  

## 汇总

|            格式            |                             说明                             |
| :------------------------: | :----------------------------------------------------------: |
|  ${string: start :length}  | 从 string 字符串的左边第 start 个字符开始，向右截取 length 个字符。 |
|      ${string: start}      |  从 string 字符串的左边第 start 个字符开始截取，直到最后。   |
| ${string: 0-start :length} | 从 string 字符串的右边第 start 个字符开始，向右截取 length 个字符。 |
|     ${string: 0-start}     |  从 string 字符串的右边第 start 个字符开始截取，直到最后。   |
|      ${string#*chars}      | 从 string 字符串第一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。 |
|     ${string##*chars}      | 从 string 字符串最后一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。 |
|      ${string%*chars}      | 从 string 字符串第一次出现 *chars 的位置开始，截取 *chars 左边的所有字符。 |
|     ${string%%*chars}      | 从 string 字符串最后一次出现 *chars 的位置开始，截取 *chars 左边的所有字符。 |

# 数组

* Shell 数组元素的下标也是从 0 开始计数。
* 获取数组中的元素要使用下标[ ]，下标可以是一个整数，也可以是一个结果为整数的表达式
* 常用的 Bash Shell 只支持一维数组

## 数组定义

在 Shell 中，用括号`( )`来表示数组，数组元素之间用空格来分隔。由此，定义数组的一般形式为：  
`array_name=(ele1  ele2  ele3 ... elen)`  

* **赋值号=两边不能有空格**
* Shell 是弱类型的，它并不要求所有数组元素的类型必须相同
  `nums=(29 100 13 8 91 44)`  
  Shell 数组的长度不是固定的，定义之后还可以增加元素。例如，对于上面的 nums 数组，它的长度是 6，使用下面的代码会在最后增加一个元素，使其长度扩展到 7：  
  `nums[6]=88`  
  下面的代码就是只给特定元素赋值：  
  `ages=([3]=24 [5]=19 [10]=12)`  
  以上代码就只给第 3、5、10 个元素赋值，所以数组长度是 3。  

## 获取数组元素

`${array_name[index]}`  
例如：  
`n=${nums[2]}`  
使用`@`或`*`可以获取数组中的所有元素，例如：  

```shell
${nums[*]}
${nums[@]}
```

## 获取数组长度

利用`@`或`*`，可以将数组扩展成列表，然后使用#来获取数组元素的个数，格式如下：

```shell
${#array_name[@]}
${#array_name[*]}
```

如果某个元素是字符串，还可以通过指定下标的方式获得该元素的长度，如下所示：`${#arr[2]}`   

## 数组拼接、合并

先利用@或*，将数组扩展成列表，然后再合并到一起。具体格式如下：  

```shell
array_new=(${array1[*]}  ${array2[*]})
```

## 数组删除

使用 unset 关键字来删除数组元素，具体格式如下：  
`unset array_name[index]`  
如果不写下标，而是写成下面的形式：  
`unset array_name`  

## 关联数组

关联数组也称为“键值对（key-value）”数组，键（key）也即字符串形式的数组下标，值（value）也即元素值。  
例如，我们可以创建一个叫做 color 的关联数组，并用颜色名字作为下标。  

```shell
declare -A color
color["red"]="#ff0000"
color["green"]="#00ff00"
color["blue"]="#0000ff"
```

也可以在定义的同时赋值：  
`declare -A color=(["red"]="#ff0000", ["green"]="#00ff00", ["blue"]="#0000ff")`  

### 访问关联数组元素

`array_name["index"]`  
加上`$()`即可获取数组元素的值：  
`$(array_name["index"])`  

### 获取元素和下标值

使用下面的形式可以获得关联数组的所有元素值：

```shell
${array_name[@]}
${array_name[*]}
```

使用下面的形式可以获取关联数组的所有下标值：

```shell
${!array_name[@]}
${!array_name[*]}
```

### 获取关联数组长度

使用下面的形式可以获得关联数组的长度：（与普通数组相同）  

```shell
${#array_name[*]}
${#array_name[@]}
```

# Shell内建命令

* 可以使用 type 来确定一个命令是否是内建命令：

```shell
[root@localhost ~]# type cd
cd is a Shell builtin
```

## alias:给命令创建别名

alisa 用来给命令创建一个别名。若直接输入该命令且不带任何参数，则列出当前 Shell 进程中使用了哪些别名。  

### 使用 alias 命令自定义别名

`alias new_name='command'`  

```shell
#!/bin/bash
alias timestamp='date +%s'
begin=`timestamp`  
sleep 20s
finish=$(timestamp)
difference=$((finish - begin))
echo "run time: ${difference}s"
```

**在代码中使用 alias 命令定义的别名只能在当前 Shell 进程中使用，在子进程和其它进程中都不能使用。当前 Shell 进程结束后，别名也随之消失。**  

### 使用 unalias 命令删除别名

使用 unalias 内建命令可以删除当前 Shell 进程中的别名。unalias 有两种使用方法：  

* 第一种用法是在命令后跟上某个命令的别名，用于删除指定的别名。
* 第二种用法是在命令后接-a参数，删除当前 Shell 进程中所有的别名。  
  **同样，这两种方法都是在当前 Shell 进程中生效的。**  

## Shell echo命令:输出字符串

用来在终端输出字符串，并在最后默认加上换行符。  

### 不换行

`echo` 命令输出结束后默认会换行，如果不希望换行，可以加上`-n`参数  

### 输出转义字符

默认情况下，echo 不会解析以反斜杠\开头的转义字符。比如，\n表示换行，echo 默认会将它作为普通字符对待。我们可以添加-e参数来让 echo 命令解析转义字符。  

* 有了-e参数，我们也可以使用转义字符\c来强制 echo 命令不换行了。

## Shell read命令：读取从键盘输入的数据

用来从标准输入中读取数据并赋值给变量。如果没有进行重定向，默认就是从键盘读取用户输入的数据；如果进行了重定向，那么可以从文件中读取数据。  
`read [-options] [variables]`  
`options`表示选项，如下表所示；`variables`表示用来存储数据的变量，可以有一个，也可以有多个。  
`options`和`variables`都是可选的，如果没有提供变量名，那么读取的数据将存放到环境变量 REPLY 中。  

|     选项     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|   -a array   |        把读取的数据赋值给数组 array，从下标 0 开始。         |
| -d delimiter | 用字符串 delimiter 指定读取结束的位置，而不是一个换行符（读取到的数据不包括 delimiter）。 |
|      -e      | 在获取用户输入的时候，对功能键进行编码转换，不会直接显示功能键对应的字符。 |
|    -n num    |              读取 num 个字符，而不是整行字符。               |
|  -p prompt   |              显示提示信息，提示内容为 prompt。               |
|      -r      |     原样读取（Raw mode），不把反斜杠字符解释为转义字符。     |
|      -s      | 静默模式（Silent mode），不会在屏幕上显示输入的字符。当输入密码和其它确认信息的时候，这是很有必要的。 |
|  -t seconds  | 设置超时时间，单位为秒。如果用户没有在指定时间内输入完成，那么 read 将会返回一个非 0 的退出状态，表示读取失败。 |
|    -u fd     | 使用文件描述符 fd 作为输入源，而不是标准输入，类似于重定向。 |


## Shell exit命令:退出当前进程

* 退出当前 Shell 进程，并返回一个退出状态；使用$?可以接收这个退出状态  
* exit 命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。  
* 一般情况下，退出状态为 0 表示成功，退出状态为非 0 表示执行失败（出错）了。
* exit 退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。
  **exit 表示退出当前 Shell 进程，我们必须在新进程中运行 test.sh，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法看到输出结果了。**  

## Shell declare和typeset命令:设置变量属性

* 建议使用 declare 代替
  declare 命令的用法如下所示：  
  `declare [+/-] [aAfFgilprtux] [变量名=变量值]`  
  其中，-表示设置属性，+表示取消属性，aAfFgilprtux都是具体的选项，它们的含义如下表所示：

|       选项       |                            含义                            |
| :--------------: | :--------------------------------------------------------: |
|    -f \[name]    |       列出之前由用户在脚本中定义的函数名称和函数体。       |
|    -F \[name]    |                   仅列出自定义函数名称。                   |
|     -g name      |              在 Shell 函数内部创建全局变量。               |
|    -p \[name]    |                  显示指定变量的属性和值。                  |
|     -a name      |                    声明变量为普通数组。                    |
|     -A name      |        声明变量为关联数组（支持索引下标为字符串）。        |
|     -i name      |                    将变量定义为整数型。                    |
| -r name\[=value] | 将变量定义为只读（不可修改和删除），等价于 readonly name。 |
| -x name\[-value] |    将变量设置为环境变量，等价于 export name\[=value]。     |

## Shell数学计算

运算符：

1. +,-,*,/,%,**
2. ++,--
3. !,&&,||
4. <,<=,>,>=,==,!=,=(= 也可以表示相当于)
5. <<,>>
6. ~,|,&,^
7. =,+=,-=,*=,/=,%=

* Shell 不能直接进行算数运算，必须使用数学计算命令

| 运算操作符/运算命令 |                             说明                             |
| :-----------------: | :----------------------------------------------------------: |
|      **(())**       |                    用于整数运算，效率很高                    |
|         let         |                 用于整数运算，和 (()) 类似。                 |
|         $[]         |                用于整数运算，不如 (()) 灵活。                |
|        expr         | 可用于整数运算，也可以处理字符串。比较麻烦，需要注意各种细节 |
|       **bc**        | Linux下的一个计算器程序，可以处理整数和小数。Shell 本身只支持整数运算，想计算小数就得使用 bc 这个外部的计算器。 |
|     declare -i      | 将变量定义为整数，然后再进行数学运算时就不会被当做字符串了。功能有限，仅支持最基本的数学运算（加减乘除和取余），不支持逻辑运算、自增自减等，所以在实际开发中很少使用。 |

### Shell (()):对整数进行数学运算

语法：
`((表达式))`  
表达式可以只有一个，也可以有多个，多个表达式之间以逗号`,`分隔。对于多个表达式的情况，以最后一个表达式的值作为整个 `(( ))` 命令的执行结果。  
可以使用`\$`获取 `(( ))` 命令的结果，这和使用`$`获得变量值是类似的。

|       用法        |                             说明                             |
| :---------------: | :----------------------------------------------------------: |
|    ((b=a-15))     | 这种写法可以在计算完成后给变量赋值。将 a-15 的运算结果赋值给变量 b |
|    c=\$((a+b))    | 可以在 `(( ))` 前面加上`\$`符号获取 `(( ))` 命令的执行结果，也即获取整个表达式的值。以 `c=\$((a+b))` 为例，即将 a+b 这个表达式的运算结果赋值给变量 c。注意，类似 `c=((a+b))` 这样的写法是错误的，不加$就不能取得表达式的结果。 |
|  ((a>7 && b==c))  |  `(( ))` 也可以进行逻辑运算，在 if 语句中常会使用逻辑运算。  |
| ((a=3+5, b=a+10)) |                  对多个表达式同时进行计算。                  |
