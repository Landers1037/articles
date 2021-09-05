---
title: shell学习笔记4
name: shell-note4
date: 2021-05-23 13:11:39
id: 0
tags: [shell]
categories: []
abstract: ""
---

 shell的字符串和数组操作


<!--more-->

 shell的字符串和数组操作

<!--more-->

## 字符串的截取

类似数组下标操作

```bash
${string: start :length}
```

## 变量截取操作

使用操作符`{}, %%, ##`

```bash
# 假设变量path
path=/dir1/dir2/dir3/file

# 删除第一个路径
path=${path#*/} # 删掉第一个/和它左边的字符
>>> dir1/dir2/dir3/file
# 删除最后一个前路径
path=${path##*/} # 删除最后一个/和它左边的字符
>>> file

# 删除第一个.和它左边的字符
path=${path#*.}

# 删除最后一个/和它右边的字符
path=${path%/*}
# 删除第一个/和它右边的字符
path=${path%%*/}
```

可以看到操作符`#`是去除左边 `%`是去除右边

在键盘上以`$`分界，`#`在左边 `%`在右边

单一符号是最小匹配 两个符号是最大匹配

## 变量替换

```bash
${path/dir1/dir2}
# 将第一个dir1替换为dir2

root@landers:/code/cooki# path=d1/d2/d3/d4
root@landers:/code/cooki# path=${path/d1/d0}
root@landers:/code/cooki# echo $path
d0/d2/d3/d4
```

```bash
${path//dir1/dir2}
# 将所有的dir1替换为dir2

root@landers:/code/cooki# path=d1/d1/d1/d2/d3
root@landers:/code/cooki# path=${path//d1/d0}
root@landers:/code/cooki# echo $path
d0/d0/d0/d2/d3
```



## 数组操作

### 创建

```bash
array=(a b c)
```

### 循环遍历

```bash
for i in ${array[@]}
do 
	echo $i
done
# 
a
b
c
```

```bash
for ((i=0;i<${#array[@]};i++))
do
	echo ${array[i]}
done
# 
a
b
c
```

使用`[@]`或者`[*]`的方式获取数组

使用`${#array[@]}`的方式获取数组的长度

使用`${array[i]}`的方式取下标为i的元素

##  字符串数组

默认情况下字符串使用空格分隔时可以解析为数组

直接对字符串进行for循环可以按照空格分隔

```bash
root@landers:/code/cooki# str="a b c"
root@landers:/code/cooki# for s in $str
> do
> echo $s
> done
a
b
c
```

但是如果我们想要使用字符串数组需要对字符串转换

```bash
root@landers:/code/cooki# str=("a b c")
root@landers:/code/cooki# echo ${#str}
5
```

此时获取到的数组长度带上了空格 这不是我们想要的

```bash
# 使用tr分隔
str="a,b,c"
array=$(echo $str | tr ',' ' ')
for i in "${!array[@]}"
do
	echo ${array[i]}
done
>>> a b c
```

```bash
# 使用正则替换
str="a,b,c"
array=(${str//,/ })
for i in "${!array[@]}"
do
	echo ${array[i]}
done
>>> 
a
b
c
```

