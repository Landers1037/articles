---
title: shell学习笔记1
name: shell-note1
date: 2020-10-16
id: 0
tags: [shell,bash]
categories: []
abstract: ""
---


Shell脚本学习笔记

<!--more-->

在编写个人项目过程中的点点积累，写出来以便以后回顾

# shell分类  

在编写之前，了解下shell的分类，我们经常说的sh脚本一般是使用shell编写的脚本

通过sh解释器来执行脚本

常见的sh解释器有`bash`, `sh`, `zsh`等

一般`bash`为大多数linux服务器自带的sh解释器

**shell脚本格式**

```bash
#!/bin/bash
# 或 #!/usr/bin/env bash
```

第一行的注释 指定了该脚本使用什么sh解释器

**执行方式**

```bash
sh xxx.sh
bash xxx.sh

# 直接运行
chmod +x xxx.sh
./xxx.sh
```

shell脚本中可以直接写运行指令或者通过shell函数的方式执行

## shell函数

使用函数定义的方式执行脚本

```bash
#!/bin/bash
function run()
{
	echo "start running"
}

run
exit
```

## shell变量

shell的变量无需其他编程语言的`var a`的形式定义

```bash
name=$1
str="hello world"
echo "$name"
echo "$str"

# 
./test.sh pig
pig
hello world

# 通过{}包裹变量以起到边界的作用
name="bell"
echo "Hello ${name}str."
# 如果不加{}做边界限定sh就会把$namestr当做变量
```

> 命名规则
>
> 同其他编程语言的命名规则
>
> - 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头
> - 中间不能有空格，可以使用下划线`_`
> - 不能使用标点符号
> - 不能使用bash的保留关键字

### 变量的语句赋值

可以通过执行语句后将语句的结果赋值给变量

```bash
data=`ls -a`
echo "$data"
```

将`ls -a`执行的结果赋值给data

> 有两种在shell里执行语句的方式
>
> ```bash
> `语句`
> ```
>
> ```bash
> $(语句)
> ```
>

### 只读变量

如果不希望变量值被修改 可以赋予它只读属性

```bash
var="hello world"
readonly var
var="new value"
# 执行会提示错误 var是只读变量
```

### 删除变量

```bash
# 使用unset命令可以删除变量
name="bell"
unset name
echo $name
```

## shell字符串

在shell脚本中最常见的就是字符串

字符串可以使用`''`或者`""`包裹，当然也可以不使用引号

> 单引号的限制
>
> 使用`''`包裹的字符串，内部会原样输出，即如果内部有$var形式的变量也不会输出变量，而是原样输出
>
> 且不能包括单个的`'`，不能带转义符
>
> 对应的
>
> 双引号`""`包裹的字符串内部可以使用变量，也可以出现转义符号

```bash
name='bell'
echo $name

word="hello world"
echo "print string :$word"
# or
echo "print string :\"$word\""
```



### 字符串的拼接

```bash
# 可以使用单引号或者双引号
str1="hello"
str2="world"
echo $str1 $str2
# opt: hello world
# 注意将两个变量并列会自动拼接

str1="bell"
str2="sra"
string1="hello "$str1""
string2="hello "$str2""
echo $string1 $string2
# 使用单引号拼接时会原样输出变量
```

### 获取字符串长度

```bash
string="abcd"
echo ${#string} #输出 4
```

### 截取字符串

```bash
string="abcdefg"
echo ${string:1:4} # 输出 bcde
# 注意起始坐标为0
```

## shell数组

数组一般也会使用

shell 数组使用`(value ...)`定义

```bash
array_name=(value0 value1 value2 value3)
# 注意没有逗号
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen
# 数组支持单独赋值 根据下标
```

**读取方式**

```bash
${数组名[下标]}
# 例如
array=(1 2 3 4)
echo ${array[0]}

# 使用@获取全部元素
echo ${array[@]}

# 使用@或者*获取长度
length=${#array_name[@]}
length=${#array_name[*]}
```

> 一般可以搭配`awk`使用
>
> 通过`awk`获取到多个数据后转换为数组，然后进行数组操作

例如：

```bash
#!/usr/bin/env bash

function test()
{
pids=(`ps ax | grep xxx | awk '{print $1}'`)
# 获取某个进程的pid或者pid列
array=${pids[@]}
# 将awk获取的列转换为数组
arraypid=($array)
firstpid=${arraypid[0]}
# 首个元素
# for循环
for p in ${arraypid[@]};do
	echo $p
done 	
}

test
exit
```



## shell判断文件存在

在shell的条件判断语句中可以直接判断文件的存在性

```bash
# 判断路径是否存在
if [ ! -d "/data/" ];then
  mkdir /data
  else
  echo "文件夹已经存在"
fi

# 判断文件是否存在
if [ -f "/data/filename" ];then
  echo "文件存在"
  else
  echo "文件不存在"
fi

-e 判断对象是否存在
-d 判断对象是否存在，并且为目录
-f 判断对象是否存在，并且为常规文件
-L 判断对象是否存在，并且为符号链接
-h 判断对象是否存在，并且为软链接
-s 判断对象是否存在，并且长度不为0
-r 判断对象是否存在，并且可读
-w 判断对象是否存在，并且可写
-x 判断对象是否存在，并且可执行
-O 判断对象是否存在，并且属于当前用户
-G 判断对象是否存在，并且属于当前用户组
```

> 注意：
>
> 当使用IO类指令比如`cd` `cat`时可能文件或者文件夹不存在
>
> cd path/xxx || exit
>
> 使用`||`操作符定义出错后退出

### exit定义退出状态码

正常情况下`exit`会返回状态码0

为了对脚本函数进行错误处理，可以自定义返回码如`exit 2333`