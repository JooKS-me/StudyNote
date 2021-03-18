## 格式

- 后缀名：建议使用`.sh`(建议是因为在Unix环境中是不需要拓展名的，加上后缀名一方面可以方便我们认出来，另一方面是告诉vim这是个shell脚本)

- 首行设置解析器类型：`#!/bin/bash`

- 注释：

  - 单行：`# 注释内容`

  - 多行：

    ```shell
    :<<!
    注释内容
    !
    ```

## 执行方式

- sh命令执行：`sh 文件名 `
- bash命令执行：`bash 文件名`
- 输入路径：`.\文件名`，这种需要执行的权限，会调用首行指定的解析器
- source命令：`source 文件名`，前面都会创建子进程，而这个不会，是直接在当前shell执行命令。

## 变量

环境变量：`env`命令即可查看

- 全局配置文件
  - /etc/profile，当shell环境初始化时，会加载全局配置文件/etc/profile里面的环境变量，供给所有shell程序使用
  - /etc/profile.d/*.sh
  - /etc/bashrc
- 个人配置文件
  - 当前用户目录/.bash_profile
  - 当前用户目录/.bashrc
- 环境变量加载初始化过程

![image-20210218175451569](https://image-1301164990.cos.ap-shanghai.myqcloud.com/img/20210218175451.png)

环境变量+自定义变量+函数：`set`命令即可查看

显示变量：`echo`

取消设定变量：`unset`

设置成环境变量：`export`，父进程给子进程传值

#### 各种环境变量

- HOME：代表用户家目录

- SHELL：当前环境使用哪个SHELL

- HISTSIZE：`history`命令记录的记录最大条数

- MAIL： 当我们使用`mail`指令收信时，系统会去读取的邮件信箱文件(mailbox)

- PATH：执行文件搜寻的路径，用冒号(:)分割

- LANG：语系数据

- RANDOM：随机数。目前大多数发行版都会有随机数生成器，即`/dev/random`，详细见下图

  ![image-20210218155640326](https://image-1301164990.cos.ap-shanghai.myqcloud.com/img/20210218155640.png)

#### 字符串

可以选择带单引号，双引号，或者啥都不带。

单引号不会显示出$变量，啥都不带不能带空格。

```shell
#左侧开始截取
${变量名:start:length} #第一个索引为0
${变量名:start}

#右侧开始截取
${变量名:0-start:length} #第一个索引为1，0后面是杠
${变量名:0-start}

#从左侧开始查找字符串，找到后，截取这个字符串后面的所有字符串
${变量名#*chars} #找第一个
${变量名##*chars} #找最后一个
#从右侧开始查找
${变量名%chars*}
${变量名%%chars*}
```

#### 自定义常量

`readonly 变量名(=value)`

`readonly -p`查看当前shell的所有常量

#### 特殊变量

- `${n}`

  当输入`sh 脚本文件名 参数1 参数2 ...`时，`$n`，获取第n个输入参数(从1开始，0为脚本文件名)

  n为二位及以上数字时需要改为`${n}`

  脚本中，echo语句输出参数时，在引号里面和外面都行

- `$#`或`${#}`

  获取所有输入参数的个数

- `$*`与`$@`，不加引号两个都一样，加了引号`$*`把参数作为一个字符串整体(单字符串)返回，`$@`把每个参数作为一个字符串返回

- `$?`返回上一条语句的执行状态，0为正常，非0表示出错了

- `$$`获取当前shell的pid，获取所有pid可以用`ps -aux`

#### 索引数组

- 定义

  ```shell
  array_name=(item1 item2 ...) #等号旁边不能有空格
  array_name=([index1]=item1 [index2]=item2)
  ```

- 获取数组的数据

  ```shell
  ${array_name[索引下标]}
  ${array_name[*]} #所有数据
  ${array_name[@]} #所有数据
  ```

- 拼接

  ```shell
  array_newname=(${array_name1[*]} ${array_name2[*]})
  ```

- 数组的删除

  ```shell
  unset array_name[索引]
  unset array_name #删掉整个数组
  ```

## 循环

```shell
for var in 参数列表
do
		# 循环体
done
```

## Shell工作环境

交互式shell与非交互式shell

```shell
交互式shell：需要用户参与互动的shell环境。
非交互式shell：只执行命令，不需要用户参与。
```

登录shell与非登录shell

```shell
登录shell环境：需要输入用户名和密码才能进入的shell环境
`sh(或bash) -l(或--login) 脚本文件名`
`或者直接键入bash -l，exit退出时会显示logout，logout退出正常`
非登录shell环境：不使用用户名和密码就能进入的shell环境
`sh(或bash) 脚本文件名`
`或者直接键入bash，exit退出时会显示exit，logout退出报错`
```

网上有个**错误**的说法，说是终端输入`echo $0`，如果是-bash表示登录环境，如果不是bash表示非登录环境。

其实并不是这样的！我在/etc/profile中新定义一个环境变量，`bash -l`可以读出这个变量，说明`bash -l`进入的的确是登录shell环境。但在`bash -l`中输入`echo $0`却显示bash，这与上面矛盾了！

所以不要简单地认为`echo $0`就能判断出当前是否为登录环境，`echo $0`在终端中只是输出当前shell解释器的名字罢了！

如果真的有需要去查询当前shell是否已经登陆的话，我会输入`ps -aux|grep bash`自上而下看，这些都是shell的进程，如果只有你只开了一个窗口，那就好说了；如果是多个窗口，或者是多个人先后登录，那就不好说了（可以让别人先退出去）。

## 文件测试运算符

### 介绍

文件测试运算符用于检测文件的各种属性。

属性检测描述如下：

| 操作符          | 说明                                                         | 举例                      |
| :-------------- | :----------------------------------------------------------- | :------------------------ |
| -b file         | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file         | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file         | directory, 检测文件是否是目录，如果是，则返回 true。         | [ -d $file ] 返回 false。 |
| -f file         | file, 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file         | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file         | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file         | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file         | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file         | read,检测文件是否可读，如果是，则返回 true。                 | [ -r $file ] 返回 true。  |
| -w file         | write,检测文件是否可写，如果是，则返回 true。                | [ -w $file ] 返回 true。  |
| -x file         | execute, 检测文件是否可执行，如果是，则返回 true。           | [ -x $file ] 返回 true。  |
| -s file         | size, 检测文件是否为空（文件大小是否大于0），不为空返回 true。 | [ -s $file ] 返回 true。  |
| -e file         | exists, 检测文件（包括目录）是否存在，如果是，则返回 true。  | [ -e $file ] 返回 true。  |
| file1 -nt file2 | new than(nt),  file1是否比file2新                            | [ file1 -nt file2 ]       |
| file1 -ot file2 | old than(ot), file1是否比file2旧                             | [ file1 -ot file2 ]       |

其他检查符：

- -S: 判断某文件是否 socket。
- -L: link, 检测文件是否存在并且是一个符号链接。

语法

```shell
[ options 文件路径字符串]
或
[[ options 文件路径字符串 ]]
```

## expr命令

### 介绍

expr 是 evaluate expressions 的缩写，译为“表达式求值”。Shell expr 是一个功能强大，并且比较复杂的命令，它除了可以实现整数计算，还可以结合一些选项对字符串进行处理，例如计算字符串长度、字符串比较、字符串匹配、字符串提取等。

### 算术语法

计算语法

```shell
expr 算术运算符表达式
```

获取计算结果赋值给新变量语法

```shell
result=`expr 算术运算符表达式`
```

### 字符串语法

计算字符串的长度语法

```shell
expr length 字符串
# 例如: expr length "itheima"  返回: 7
```

截取字符串语法

```shell
expr substr 字符串 start end
# start 截取字符串的起始位置, 从1开始
# end 截取字符串的结束位置, 包含这个位置截取
# 例如 expr substr "itheima" 1 2  返回: it
```

获取第一个字符在字符串中出现的位置语法

```shell
expr index 被查找字符串  需要查找的字符
# 例如 expr index "itheima" t  会返回: 2 
```

正则表达式匹配1语法

```shell
expr match 字符串 正则表达式
# 正则表达式默认带有^ ,  代表以什么开头
# 返回值为符合匹配字符的长度, 否则返回为0
# 例如: expr match "itheima" ".*m"  会返回: 6
# 正则表达式通配符"."代表任意一个字符
# 正则表达式通配符"*"代表前面的字符可以出现0到多次
# ".*m" 含义为 匹配字符串中m前面的字符串长度！ 
```

正则表表达式匹配2语法,  功能与语法1一样

```shell
expr 字符串 : 正则表达式
# 正则表达式默认带有^ ,  代表以什么开头
# 返回值为符合匹配字符的长度, 否则返回为0
# 例如: expr "itheima" : ".*m"  会返回: 6
```

## 流程控制语句：if else语句

### 目标

能够使用if条件语句进行条件判断

### 介绍

if条件判断逻辑控制语句

### if语法

多行写法语法

```shell
if  条件
then
    命令
fi
```

> 可以将if语句放入一行语法
>
> ```shell
> if 条件; then 命令; fi
> ```

### if else 语法

```shell
if  条件
then
   命令
else
   命令
fi
```

### if elif else 语法

```shell
if  条件1
then
   命令1
elif 条件2
then
    命令2
elif 条件3
then
    命令3
……
else
   命令N
fi
```

## *流程控制语句：case语句

### 介绍

Shell case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令;

当分支较多，并且判断条件比较简单时，使用 case in 语句就比较方便了。

### 语法

```shell
case 值 in
匹配模式1)
    命令1
    命令2
    ...
    ;;
匹配模式2）
    命令1
    命令2
    ...
    ;;
*)
    命令1
    命令2
    ...
    ;;
esac
```

每一匹配模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;(类似break, 不可以替代否则语法报错)。取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

case、in 和 esac 都是 Shell 关键字,   esac就是case的反写在这里代表结束case

匹配模式:  可以是一个数字、一个字符串，甚至是一个简单正则表达式。

简单正则表达式支持如下通配符

| 格式  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| *     | 表示任意字符串。                                             |
| [abc] | 表示 a、b、c 三个字符中的任意一个。比如，[15ZH] 表示 1、5、Z、H 四个字符中的任意一个。 |
| [m-n] | 表示从 m 到 n 的任意一个字符。比如，[0-9] 表示任意一个数字，[0-9a-zA-Z] 表示字母或数字。 |
| \|    | 表示多重选择，类似逻辑运算中的或运算。比如，abc \| xyz 表示匹配字符串 "abc" 或者 "xyz"。 |

## 流程控制：while语句

### 介绍

while用于循环执行一系列命令



### 语法

多行写法

```shell
while 条件
do
	命令1
	命令2
	...
	continue; # 结束当前这一次循环, 进入下一次循环
	break; # 结束当前循环
done
```

一行写法

```shell
while 条件; do 命令; done;
```

## *流程控制：until语句

### 介绍

until 也是循环结构语句,  until 循环与 while 循环在处理方式上刚好相反,  循环条件为false会一致循环, 条件为true停止循环.

### 语法

```shell
until 条件
do
    命令
done
```

条件如果返回值为1(代表false)，则继续执行循环体内的语句，否则跳出循环。

## 流程控制：for语句

### 介绍

Shell支持for循环,  与其他编程语言类似.

### 循环方式1

#### 语法

多行写法

```shell
for var in item1 item2 ... itemN
do
    命令1
    命令2
    ...
done
```

一行写法

```shell
for var in item1 item2 ... itemN; do 命令1; 命令2…; done;
```

> var是循环变量
>
> item1 item2 ... itemN 是循环的范围

### 循环方式2

#### 语法

多行写法

```shell
for var in {start..end}
do
	命令
done
```

> start:  循环范围的起始值,必须为整数
>
> end: 循环范围的结束值, 必须为整数

一行写法

```shell
for var in {start..end}; do 命令; done
```

### 循环方式3

#### 语法

多行写法

```shell
for((i=start;i<=end;i++))
do
	命令
done
```

一行写法

```shell
for((i=start;i<=end;i++)); do 命令; done
```

## 流程控制：select语句

### 介绍

select in 循环用来增强交互性，它可以显示出带编号的菜单，用户输入不同的编号就可以选择不同的菜单，并执行不同的功能.   select in 是 Shell 独有的一种循环，非常适合终端（Terminal）这样的交互场景, 其他语言没有;

### 语法

```shell
select var in menu1 menu2 ...
do
    命令
done
```

> 注意: select 是无限循环（死循环），输入空值，或者输入的值无效，都不会结束循环，只有遇到 break 语句，或者按下 Ctrl+D 组合键才能结束循环。
>
> 执行命令过程中: 终端会输出 `#?`  代表可以输入选择的菜单编号