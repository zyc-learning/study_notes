# Shell脚本编程

脚本执行方法：

使用 `./test.sh`：这种方式会尝试以独立的进程运行脚本，但需要脚本具有执行权限。

使用 `. test.sh`（点命令）：这种方式是将脚本的内容“源”到当前Shell环境中执行，不需要脚本具有执行权限，但需要脚本具有读权限。



## 变量

### 变量命名

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用 bash 里的关键字（可用 help 命令查看保留关键字）。



### 声明变量

访问变量的语法形式为：`${var}` 和 `$var` 。

变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，所以推荐加花括号。

```shell
#!/bin/bash
word="hello"
echo ${word}
# Output: hello

#!/bin/bash 是一个 shebang（也称为 hashbang 或 hashpling），它在脚本文件的第一行中使用，用于指定脚本的解释器。
作用：
指定解释器：告诉操作系统使用 /bin/bash（Bash Shell）来执行这个脚本。
确保兼容性：即使在不同的系统环境中，也能保证脚本按照预期的方式运行。
可移植性：使脚本可以在任何支持 Bash 的系统上运行，而无需手动指定解释器。
```



### 只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。

```bash
#!/bin/bash
rword="hello"
echo ${rword}
readonly rword
# rword="bye"  # 如果放开注释，执行时会报错
```



### 删除变量

```bash
dword="hello"  # 声明变量
echo ${dword}  # 输出变量值
# Output: hello

unset dword    # 删除变量
echo ${dword}
# Output: （空）
```



### 变量类型

- **局部变量** - 局部变量是仅在某个脚本内部有效的变量。它们不能被其他的程序和脚本访问。
- **环境变量** - 环境变量是对当前 shell 会话内所有的程序或脚本都可见的变量。创建它们跟创建局部变量类似，但使用的是 `export` 关键字，shell 脚本也可以定义环境变量。

**常见的环境变量：**

```shell
Shell常见的变量之二环境变量，主要是在程序运行时需要设置，环境变量详解如下：

PATH  		命令所示路径，以冒号为分割；
HOME  		打印用户家目录；
SHELL 		显示当前Shell类型；
USER  		打印当前用户名；
ID    		打印当前用户id信息；
PWD   		显示当前所在路径；
TERM  		打印当前终端类型；
HOSTNAME    显示当前主机名；
PS1         定义主机命令提示符的；
HISTSIZE    历史命令大小，可通过 HISTTIMEFORMAT 变量设置命令执行时间;
RANDOM      随机生成一个 0 至 32767 的整数;
HOSTNAME    主机名
```

- **本地变量** - 生效范围仅为当前shell进程；（其他shell，当前的子sehll进程均无效）

  - 变量赋值：name = “value”

- **位置变量** - shell 脚本中用来引用命令行参数的特殊变量。当你运行一个 shell 脚本时,可以在命令行上传递参数,这些参数可以在脚本中使用位置变量引用。

  位置变量包括以下几种:

  1. `$0`: 表示脚本本身的名称。
  2. `$1`, `$2`, `$3`, ..., `$n`: 分别表示第1个、第2个、第3个...第n个参数。
  3. `$#`: 表示传递给脚本的参数个数。
  4. `$*`: 表示所有参数,将所有参数当作一个整体。
  5. `$@`: 表示所有参数,但是每个参数都是独立的。



## 字符串

shell 字符串可以用单引号 `' '`，也可以用双引号 `" "`，也可以不用引号（一般不会不加引号）。

单引号的特点：单引号里不识别变量、单引号里不能出现单独的单引号（使用转义符也不行），但可成对出现，作为字符串拼接使用。

双引号的特点：双引号里识别变量、双引号里可以出现转义字符



### 字符串的拼接

```bash
# 使用单引号拼接
name1='white'
str1='hello, '${name1}''
str2='hello, ${name1}'
echo ${str1}_${str2}
# Output:
# hello, white_hello, ${name1}

# 使用双引号拼接
name2="black"
str3="hello, "${name2}""
str4="hello, ${name2}"
echo ${str3}_${str4}
# Output:
# hello, black_hello, black
```

### 获取字符串的长度

```bash
text="12345"
echo ${#text}

# Output:
# 5
```

### 截取子字符串

```bash
${variable:start:length}
text="12345"
echo ${text:2:2}

# Output:
# 34
```



## 数组

bash 只支持一维数组。数组下标从 0 开始，下标可以是整数或算术表达式，其值应大于或等于 0。(类似python)

### 创建/访问数组

```bash
array_name=(value1 value2 value3 ...)
array_name=([0]=value1 [1]=value2 ...)

# 案例一
[root@localhost ~]# cat a.sh
#!/bin/bash
# 创建数组
fruits=("apple" "banana" "orange")

# 访问元素
echo "First fruit: ${fruits[0]}"
echo "All fruits: ${fruits[@]}"

[root@localhost ~]# bash a.sh
First fruit: apple
All fruits: apple banana orange
```

访问数组中所有的元素：

```bash
[root@localhost ~]# cat a.sh
nums=([0]="nls" [1]="18" [2]="teacher")
echo ${nums[*]}
echo ${nums[@]}

[root@localhost ~]# bash a.sh
nls 18 teacher
nls 18 teacher
```

### 获取数组的长度

```bash
[root@localhost ~]# cat a.sh
nums=([0]="nls" [1]="18" [2]="teacher")
echo "数组元素个数为: ${#nums[*]}"

[root@localhost ~]# bash a.sh
数组元素个数为: 3
```

### 删除元素

用`unset`命令来从数组中删除一个元素：

```bash
[root@localhost ~]# cat a.sh
nums=([0]="nls" [1]="18" [2]="teacher")
echo "数组元素个数为: ${#nums[*]}"
unset nums[0]
echo "数组元素个数为: ${#nums[*]}"

[root@localhost ~]# bash a.sh
数组元素个数为: 3
数组元素个数为: 2
```



## 运算符

### 算数运算符

| 运算符 | 说明                                          | 举例                           |
| ------ | --------------------------------------------- | ------------------------------ |
| +      | 加法                                          | `expr $x + $y` 结果为 30。     |
| -      | 减法                                          | `expr $x - $y` 结果为 -10。    |
| *      | 乘法                                          | `expr $x * $y` 结果为 200。    |
| /      | 除法                                          | `expr $y / $x` 结果为 2。      |
| %      | 取余                                          | `expr $y % $x` 结果为 0。      |
| =      | 赋值                                          | `x=$y` 将把变量 y 的值赋给 x。 |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | `[ $x == $y ]` 返回 false。    |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | `[ $x != $y ]` 返回 true。     |

**注意：**条件表达式要放在方括号之间，并且要有空格，例如: `[$x==$y]` 是错误的，必须写成 `[ $x == $y ]`

**示例：**

- expr本身是一个命令，可以直接进行运算

```bash
x=10
y=20

echo "x=${x}, y=${y}"

val=`expr ${x} + ${y}`
echo "${x} + ${y} = $val"

val=`expr ${x} - ${y}`
echo "${x} - ${y} = $val"

val=`expr ${x} \* ${y}`
echo "${x} * ${y} = $val"

val=`expr ${y} / ${x}`
echo "${y} / ${x} = $val"

val=`expr ${y} % ${x}`
echo "${y} % ${x} = $val"

if [[ ${x} == ${y} ]]
then
  echo "${x} = ${y}"
fi
if [[ ${x} != ${y} ]]
then
  echo "${x} != ${y}"
fi

#  Execute: ./operator-demo.sh
#  Output:
#  x=10, y=20
#  10 + 20 = 30
#  10 - 20 = -10
#  10 * 20 = 200
#  20 / 10 = 2
#  20 % 10 = 0
#  10 != 20
```



### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

| 运算符 | 说明                                                  | 举例                         |
| ------ | ----------------------------------------------------- | ---------------------------- |
| `-eq`  | 检测两个数是否相等，相等返回 true。                   | `[ $a -eq $b ]`返回 false。  |
| `-ne`  | 检测两个数是否相等，不相等返回 true。                 | `[ $a -ne $b ]` 返回 true。  |
| `-gt`  | 检测左边的数是否大于右边的，如果是，则返回 true。     | `[ $a -gt $b ]` 返回 false。 |
| `-lt`  | 检测左边的数是否小于右边的，如果是，则返回 true。     | `[ $a -lt $b ]` 返回 true。  |
| `-ge`  | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ]` 返回 false。 |
| `-le`  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]`返回 true。   |

**示例：**

```bash
x=10
y=20

echo "x=${x}, y=${y}"

if [[ ${x} -eq ${y} ]]; then
   echo "${x} -eq ${y} : x 等于 y"
else
   echo "${x} -eq ${y}: x 不等于 y"
fi

if [[ ${x} -ne ${y} ]]; then
   echo "${x} -ne ${y}: x 不等于 y"
else
   echo "${x} -ne ${y}: x 等于 y"
fi

if [[ ${x} -gt ${y} ]]; then
   echo "${x} -gt ${y}: x 大于 y"
else
   echo "${x} -gt ${y}: x 不大于 y"
fi

if [[ ${x} -lt ${y} ]]; then
   echo "${x} -lt ${y}: x 小于 y"
else
   echo "${x} -lt ${y}: x 不小于 y"
fi

if [[ ${x} -ge ${y} ]]; then
   echo "${x} -ge ${y}: x 大于或等于 y"
else
   echo "${x} -ge ${y}: x 小于 y"
fi

if [[ ${x} -le ${y} ]]; then
   echo "${x} -le ${y}: x 小于或等于 y"
else
   echo "${x} -le ${y}: x 大于 y"
fi

#  Execute: ./operator-demo2.sh
#  Output:
#  x=10, y=20
#  10 -eq 20: x 不等于 y
#  10 -ne 20: x 不等于 y
#  10 -gt 20: x 不大于 y
#  10 -lt 20: x 小于 y
#  10 -ge 20: x 小于 y
#  10 -le 20: x 小于或等于 y
```



### 字符串运算符

| 运算符 | 说明                                       | 举例                       |
| ------ | ------------------------------------------ | -------------------------- |
| `=`    | 检测两个字符串是否相等，相等返回 true。    | `[ $a = $b ]` 返回 false。 |
| `!=`   | 检测两个字符串是否相等，不相等返回 true。  | `[ $a != $b ]` 返回 true。 |
| `-z`   | 检测字符串长度是否为 0，为 0 返回 true。   | `[ -z $a ]` 返回 false。   |
| `-n`   | 检测字符串长度是否为 0，不为 0 返回 true。 | `[ -n $a ]` 返回 true。    |
| `str`  | 检测字符串是否为空，不为空返回 true。      | `[ $a ]` 返回 true。       |

示例：

```bash
x="abc"
y="xyz"


echo "x=${x}, y=${y}"

if [[ ${x} = ${y} ]]; then
   echo "${x} = ${y} : x 等于 y"
else
   echo "${x} = ${y}: x 不等于 y"
fi

if [[ ${x} != ${y} ]]; then
   echo "${x} != ${y} : x 不等于 y"
else
   echo "${x} != ${y}: x 等于 y"
fi

if [[ -z ${x} ]]; then
   echo "-z ${x} : 字符串长度为 0"
else
   echo "-z ${x} : 字符串长度不为 0"
fi

if [[ -n "${x}" ]]; then
   echo "-n ${x} : 字符串长度不为 0"
else
   echo "-n ${x} : 字符串长度为 0"
fi

if [[ ${x} ]]; then
   echo "${x} : 字符串不为空"
else
   echo "${x} : 字符串为空"
fi

#  Execute: ./operator-demo5.sh
#  Output:
#  x=abc, y=xyz
#  abc = xyz: x 不等于 y
#  abc != xyz : x 不等于 y
#  -z abc : 字符串长度不为 0
#  -n abc : 字符串长度不为 0
#  abc : 字符串不为空
```



### 逻辑运算符

| 运算符 | 说明       | 举例                                            |           |                                               |
| ------ | ---------- | ----------------------------------------------- | --------- | --------------------------------------------- |
| `&&`   | 逻辑的 AND | `[[ ${x} -lt 100 && ${y} -gt 100 ]]` 返回 false |           |                                               |
| `      |            | `                                               | 逻辑的 OR | `[[ ${x} -lt 100 && ${y} -gt 100 ]]`返回 true |

示例：

```bash
x=10
y=20

echo "x=${x}, y=${y}"

if [[ ${x} -lt 100 && ${y} -gt 100 ]]
then
   echo "${x} -lt 100 && ${y} -gt 100 返回 true"
else
   echo "${x} -lt 100 && ${y} -gt 100 返回 false"
fi

if [[ ${x} -lt 100 || ${y} -gt 100 ]]
then
   echo "${x} -lt 100 || ${y} -gt 100 返回 true"
else
   echo "${x} -lt 100 || ${y} -gt 100 返回 false"
fi

#  Execute: ./operator-demo4.sh
#  Output:
#  x=10, y=20
#  10 -lt 100 && 20 -gt 100 返回 false
#  10 -lt 100 || 20 -gt 100 返回 true
```



### 布尔运算符

| 运算符 | 说明                                                | 举例                                       |
| ------ | --------------------------------------------------- | ------------------------------------------ |
| `!`    | 非运算，表达式为 true 则返回 false，否则返回 true。 | `[ ! false ]` 返回 true。                  |
| `-o`   | 或运算，有一个表达式为 true 则返回 true。           | `[ $a -lt 20 -o $b -gt 100 ]` 返回 true。  |
| `-a`   | 与运算，两个表达式都为 true 才返回 true。           | `[ $a -lt 20 -a $b -gt 100 ]` 返回 false。 |

示例：

```bash
x=10
y=20

echo "x=${x}, y=${y}"

if [[ ${x} != ${y} ]]; then
   echo "${x} != ${y} : x 不等于 y"
else
   echo "${x} != ${y}: x 等于 y"
fi

if [[ ${x} -lt 100 && ${y} -gt 15 ]]; then
   echo "${x} 小于 100 且 ${y} 大于 15 : 返回 true"
else
   echo "${x} 小于 100 且 ${y} 大于 15 : 返回 false"
fi

if [[ ${x} -lt 100 || ${y} -gt 100 ]]; then
   echo "${x} 小于 100 或 ${y} 大于 100 : 返回 true"
else
   echo "${x} 小于 100 或 ${y} 大于 100 : 返回 false"
fi

if [[ ${x} -lt 5 || ${y} -gt 100 ]]; then
   echo "${x} 小于 5 或 ${y} 大于 100 : 返回 true"
else
   echo "${x} 小于 5 或 ${y} 大于 100 : 返回 false"
fi

#  Execute: ./operator-demo3.sh
#  Output:
#  x=10, y=20
#  10 != 20 : x 不等于 y
#  10 小于 100 且 20 大于 15 : 返回 true
#  10 小于 100 或 20 大于 100 : 返回 true
#  10 小于 5 或 20 大于 100 : 返回 false
```



### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

属性检测描述如下：

| 操作符  | 说明                                                         | 举例                        |
| ------- | ------------------------------------------------------------ | --------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | `[ -b $file ]` 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | `[ -c $file ]` 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | `[ -d $file ]` 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | `[ -f $file ]` 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | `[ -g $file ]` 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | `[ -k $file ]`返回 false。  |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | `[ -p $file ]` 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | `[ -u $file ]` 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | `[ -r $file ]` 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | `[ -w $file ]` 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | `[ -x $file ]` 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于 0），不为空返回 true。    | `[ -s $file ]` 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | `[ -e $file ]` 返回 true。  |

**⌨️ 『示例源码』** [operator-demo6.sh](https://github.com/dunwu/os-tutorial/blob/master/codes/shell/demos/operator/operator-demo6.sh)

```bash
file="/etc/hosts"

if [[ -r ${file} ]]; then
   echo "${file} 文件可读"
else
   echo "${file} 文件不可读"
fi
if [[ -w ${file} ]]; then
   echo "${file} 文件可写"
else
   echo "${file} 文件不可写"
fi
if [[ -x ${file} ]]; then
   echo "${file} 文件可执行"
else
   echo "${file} 文件不可执行"
fi
if [[ -f ${file} ]]; then
   echo "${file} 文件为普通文件"
else
   echo "${file} 文件为特殊文件"
fi
if [[ -d ${file} ]]; then
   echo "${file} 文件是个目录"
else
   echo "${file} 文件不是个目录"
fi
if [[ -s ${file} ]]; then
   echo "${file} 文件不为空"
else
   echo "${file} 文件为空"
fi
if [[ -e ${file} ]]; then
   echo "${file} 文件存在"
else
   echo "${file} 文件不存在"
fi

#  Execute: ./operator-demo6.sh
#  Output:(根据文件的实际情况，输出结果可能不同)
#  /etc/hosts 文件可读
#  /etc/hosts 文件可写
#  /etc/hosts 文件不可执行
#  /etc/hosts 文件为普通文件
#  /etc/hosts 文件不是个目录
#  /etc/hosts 文件不为空
#  /etc/hosts 文件存在
```



### 用户交互read

常用选项

| 选项 | 描述                       |
| :--- | :------------------------- |
| `-p` | 在读取输入之前显示提示信息 |
| `-n` | 限制输入的字符数           |
| `-s` | 隐藏用户输入               |
| `-a` | 将输入存储到数组变量中     |
| `-d` | 指定用于终止输入的分隔符   |
| `-t` | 设置超时时间(以秒为单位)   |
| `-e` | 允许使用 Readline 编辑键   |
| `-i` | 设置默认值                 |



## 控制语句

### 条件语句

  跟其它程序设计语言一样，Bash 中的条件语句让我们可以决定一个操作是否被执行。结果取决于一个包在`[[ ]]`里的表达式。由`[[ ]]`（`sh`中是`[ ]`）包起来的表达式被称作 **检测命令** 或 **基元**。这些表达式帮助我们检测一个条件的结果

1. `if` 语句

`if`在使用上跟其它语言相同。如果中括号里的表达式为真，那么`then`和`fi`之间的代码会被执行。`fi`标志着条件代码块的结束。

```bash
# 写成一行
if [[ 1 -eq 1 ]]; then echo "1 -eq 1 result is: true"; fi
# Output: 1 -eq 1 result is: true

# 写成多行
if [[ "abc" -eq "abc" ]]
then
  echo ""abc" -eq "abc" result is: true"
fi
# Output: abc -eq abc result is: true
```

`if else` 语句

```bash
if [[ 2 -ne 1 ]]; then
  echo "true"
else
  echo "false"
fi
# Output: true
```

`if elif else` 语句

```bash
x=10
y=20
if [[ ${x} > ${y} ]]; then
   echo "${x} > ${y}"
elif [[ ${x} < ${y} ]]; then
   echo "${x} < ${y}"
else
   echo "${x} = ${y}"
fi
# Output: 10 < 20
```



### 循环语句

循环其实不足为奇。跟其它程序设计语言一样，bash 中的循环也是只要控制条件为真就一直迭代执行的代码块。Bash 中有四种循环：`for`，`while`，`until`和`select`。

#### for循环

`for`与 C 语言中非常像。看起来是这样：

```bash
for arg in elem1 elem2 ... elemN
do
  ### 语句
done
```

在每次循环的过程中，`arg`依次被赋值为从`elem1`到`elemN`。这些值还可以是通配符或者[大括号扩展]

当然，我们还可以把`for`循环写在一行，但这要求`do`之前要有一个分号，就像下面这样：

```bash
for i in {1..5}; do echo $i; done
```

还有，如果你觉得`for..in..do`对你来说有点奇怪，那么你也可以像 C 语言那样使用`for`，比如：

```bash
for (( i = 0; i < 10; i++ )); do
  echo $i
done
```

当我们想对一个目录下的所有文件做同样的操作时，`for`就很方便了。举个例子，如果我们想把所有的`.bash`文件移动到`script`文件夹中，并给它们可执行权限，我们的脚本可以这样写：

```bash
DIR=/home/zp
for FILE in ${DIR}/*.sh; do
  mv "$FILE" "${DIR}/scripts"
done
# 将 /home/zp 目录下所有 sh 文件拷贝到 /home/zp/scripts
```

##### 案例一：创建用户

创建用户user1‐user10家目录，并且在user1‐10家目录下创建1.txt‐10.txt

```bash
[root@localhost ~]# cat adduser.sh
#!/bin/bash
for i in {1..10}
do
    mkdir /home/user$i
    for j in $(seq 10)
    do
       touch /home/user$i/$j.txt
    done
done

# Output:
[root@localhost ~]# bash adduser.sh
[root@localhost ~]# ls /home/
user01  user10  user3  user5  user7  user9
user1   user2   user4  user6  user8
[root@localhost ~]# ls /home/user1
10.txt  2.txt  4.txt  6.txt  8.txt
1.txt   3.txt  5.txt  7.txt  9.txt
```



#### while循环

`while`循环检测一个条件，只要这个条件为 *真*，就执行一段命令。被检测的条件跟`if..then`中使用的[基元](https://github.com/denysdovhan/bash-handbook/blob/master/translations/zh-CN/README.md#基元和组合表达式)并无二异。因此一个`while`循环看起来会是这样：

```bash
while 循环条件
do
  ### 语句
done
```

##### 案例一：数字累加

计算1+2+..10的总和

```bash
[root@localhost ~]# cat sum.sh
#!/bin/bash
i=1
sum=0
while [ $i -lt 10 ]
do
    let sum+=$i
    let i++
done
echo $sum

# Output:
[root@localhost ~]# bash sum.sh
45
```

##### 案例二：猜数字小游戏

加上随机数

```bash
[root@localhost ~]# cat guess.sh
#!/bin/bash
num2=$((RANDOM%100+1))
while true
do
    read -p "请输入你要猜的数字：" num1
    if [ $num1 -gt $num2 ];then
        echo "你猜大了"
    elif [ $num1 -lt $num2 ];then
        echo "你猜小了"
    else
        echo "你猜对了"
        break
    fi
done

# Output:
[root@localhost ~]# bash guess.sh
请输入你要猜的数字：50
你猜小了
请输入你要猜的数字：70
你猜小了
请输入你要猜的数字：90
你猜大了
请输入你要猜的数字：80
你猜大了
```



#### until循环

`until`循环跟`while`循环正好相反。它跟`while`一样也需要检测一个测试条件，但不同的是，只要该条件为 *假* 就一直执行循环：

```bash
until 条件测试
do
    ##循环体
done
```

示例：

```bash
[root@localhost ~]# cat until.sh
x=0
until [ ${x} -ge 5 ]; do
  echo ${x}
  x=`expr ${x} + 1`
done

# Output
[root@localhost ~]# bash until.sh
0
1
2
3
4
```



#### 退出循环

break和continue

示例：

```shell
# 查找 10 以内第一个能整除 2 和 3 的正整数
i=1
while [[ ${i} -lt 10 ]]; do
  if [[ $((i % 3)) -eq 0 ]] && [[ $((i % 2)) -eq 0 ]]; then
    echo ${i}
    break;
  fi
  i=`expr ${i} + 1`
done

# Output: 6

# 打印10以内的奇数
for (( i = 0; i < 10; i ++ )); do
  if [[ $((i % 2)) -eq 0 ]]; then
    continue;
  fi
  echo ${i}
done

#  Output:
#  1
#  3
#  5
#  7
#  9
```



## 函数

### 函数定义

bash 函数定义语法如下：

```bash
[ function ] funname [()] {
    action;
    [return int;]
}
function FUNNAME(){
函数体
返回值
}
FUNNME #调用函数
```

 说明：

 1. 函数定义时，`function` 关键字可有可无。
 2. 函数返回值 - return 返回函数返回值，返回值类型只能为整数（0-255）。如果不加 return 语句，shell 默认将以最后一条命令的运行结果，作为函数返回值。
 3. 函数返回值在调用该函数后通过 `$?` 来获得。
 4. 所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至 shell 解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。



示例：

```bash
[root@localhost ~]# cat func.sh
#!/bin/bash
func(){
    echo "这是我的第一个函数"
}

echo "------函数执行之前-------"
func
echo "------函数执行之前-------"

# Output:
[root@localhost ~]# bash func.sh
------函数执行之前-------
这是我的第一个函数
------函数执行之前-------
```

### 返回值

示例：

```bash
func(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
func
echo "输入的两个数字之和为 $? !"
#可以使用$?来获取返回值
```



### 函数参数

**位置参数**是在调用一个函数并传给它参数时创建的变量

| 变量           | 描述                           |      |
| -------------- | ------------------------------ | ---- |
| `$0`           | 脚本名称                       |      |
| `$1 … $9`      | 第 1 个到第 9 个参数列表       |      |
| `${10} … ${N}` | 第 10 个到 N 个参数列表        |      |
| `$*` or `$@`   | 除了`$0`外的所有位置参数       |      |
| `$#`           | 不包括`$0`在内的位置参数的个数 |      |
| `$FUNCNAME`    | 函数名称（仅在函数内部有值）   |      |

示例：

```bash
#!/bin/bash

x=0
if [[ -n $1 ]]; then
  echo "第一个参数为：$1"
  x=$1
else
  echo "第一个参数为空"
fi

y=0
if [[ -n $2 ]]; then
  echo "第二个参数为：$2"
  y=$2
else
  echo "第二个参数为空"
fi

paramsFunction(){
  echo "函数第一个入参：$1"
  echo "函数第二个入参：$2"
}
paramsFunction ${x} ${y}
```

执行结果：

```bash
[root@localhost ~]# vim func1.sh
[root@localhost ~]# bash func1.sh
第一个参数为空
第二个参数为空
函数第一个入参：0
函数第二个入参：0
[root@localhost ~]# bash func1.sh 10 20
第一个参数为：10
第二个参数为：20
函数第一个入参：10
函数第二个入参：20
```

### 函数处理参数

另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                             |
| -------- | ------------------------------------------------ |
| `$#`     | 返回参数个数                                     |
| `$*`     | 返回所有参数                                     |
| `$       | 参数处理                                         |
| -------- | ------------------------------------------------ |

| `$!` | 后台运行的最后一个进程的 ID 号 | | `$@` | 返回所有参数 | | `$-` | 返回 Shell 使用的当前选项，与 set 命令功能相同。 | | `$?` | 函数返回值 |

```bash
runner() {
  return 0
}

name=zp
paramsFunction(){
  echo "函数第一个入参：$1"
  echo "函数第二个入参：$2"
  echo "传递到脚本的参数个数：$#"
  echo "所有参数："
  printf "+ %s\n" "$*"
  echo "脚本运行的当前进程 ID 号：$$"
  echo "后台运行的最后一个进程的 ID 号：$!"
  echo "所有参数："
  printf "+ %s\n" "$@"
  echo "Shell 使用的当前选项：$-"
  runner
  echo "runner 函数的返回值：$?"
}
paramsFunction 1 "abc" "hello, \"zp\""
#  Output:
#  函数第一个入参：1
#  函数第二个入参：abc
#  传递到脚本的参数个数：3
#  所有参数：
#  + 1 abc hello, "zp"
#  脚本运行的当前进程 ID 号：26400
#  后台运行的最后一个进程的 ID 号：
#  所有参数：
#  + 1
#  + abc
#  + hello, "zp"
#  Shell 使用的当前选项：hB
#  runner 函数的返回值：0
```
