---
title: go中指针的使用
name: go-base-pointer
date: 2020-03-16
id: 0
tags: [go]
categories: []
abstract: ""
---


Go中指针的使用


<!--more-->


Go中指针的使用

<!--more-->

## 指针

使用`&`和`*`这对指针操作符可以对指针进行操作，`&`可以获取变量的内存地址，`*`可以从该指针指向的地址获取值

### 定义指针

```go
var p *int
var a int
a = 10
p = &a
fmt.Print(a)
```

结果

```go
10 0xc000064080
```

p指针存储的是a变量的地址

```go
	var p *int
	var a int
	a = 10
	p = &a
	fmt.Print(a,*p)
```

结果

```go
10 10
```

使用`*`取出p指向区域的值

### 数值互换

```go
	var x,y int
	x,y = 1,2
	fmt.Println("交换内存地址下的值")
	px ,py := &x,&y
	tmp1 := *px
	*px = *py
	*py = tmp1
	fmt.Printf("%d%d\n",x,y)
	
	//仅仅交换内存地址
	var i ,j int
	i,j = 1,2
	fmt.Println("交换内存地址")
	pi,pj := &i,&j
	tmp2 := pi
	pi = pj
	pj = tmp2
	fmt.Printf("%d%d\n",i,j)
```

结果

```go
2 1
1 2
```

第一部分，我们传入指针然后交换指针指向的值

第二部分，我们传入指针，但是只互换指针

可以看到只有第一部分完成了数值的交换，第二部分保持不变因为指针只是指向一片内存区域。仅仅交换指针但是指针下对应的数据仍保持 不变

### 使用New定义指针

```go
p := new(int)
var a int = 1
p = &a
fmt.Print(*p)
```

结果

```go
1
```

### 其他类型指针

**数组指针**

```go
var p *[]int
array := []int{1,2,3}
p = &array
fmt.Print(*p)
```

结果

```go
[1 2 3]
```

**切片**

切片的取值是使用指针完成的

```go
	var arr1 = []int{1,2,3}
	a1 := arr1[:]
	a2 := arr1[1:len(arr1)]

	fmt.Println(a1,a2)
```

结果

```go
[1 2 3] [2 3]
```

