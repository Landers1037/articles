---
title: go的函数声明
name: go-base-func
date: 2020-03-16
id: 0
tags: [go]
categories: []
abstract: ""
---


Go的函数

<!--more-->

 函数定义分为三类，普通函数，匿名函数，方法

### 普通函数

即包含函数名，参数，返回值的函数体

```go
func print(s string) string{
    fmt.Print("input is",s)
    return s
}
```

函数名`print`后的()内是传入的参数，后一个是返回值。当有多个返回值时可以使用`(data1,data2 string)`括号的形式

如果仅仅指定了返回值类型如`string`，那么return后面必须添加返回的变量

如果在返回值里定义了变量名如`(data string)`，那么结尾只需`return`

### 匿名函数

```go
func (){
    ...
}
```

匿名函数可以有返回值，可以在定义后赋值给一个函数变量

```go
	f2 := func(data int) int{
		return data+1
	}(1)

	fmt.Print(f2)
```

此时的f2就是后面的匿名函数传入参数`2`后的值，定义匿名函数后可以在`}`后使用`()`传入参数立刻实例化

```go
	f := func(i int) {
		fmt.Printf("input is %d\n",i)
	}
	f(2)
```

也可以把匿名函数赋值给函数变量f，执行`f()`时传入参数就会实例化。此时的f实际是一个函数类型的变量

```go
var f func()
```

### 匿名函数作为回调

什么是回调?执行函数后该函数触发的操作

```go
func show(list []int,f func(int))  {
	for _,v := range list{
		f(v)
	}
}

func main()  {
	//回调
	show([]int{1,2,3}, func(i int) {
		fmt.Print(i)
	})
}
```

在show里定义了一个匿名函数f，在调用`show`时这个匿名函数就会把传入的操作实例化

匿名函数还可以用于封装函数

```go
switch{
    case 1:func(){}
}
```

### 接口

函数可以作为go中的接口使用

**定义一个接口，其实现了call方法**

```go
type Test interface {
	call(s string)
}

type TestChild struct {

}
func (t TestChild) call(s string) {
	fmt.Println("this is an interface by child")
	fmt.Println("what call is",s)
}

func main()  {
	var t TestChild
	t.call("hello")
}
```

我们定义了一个接口`Test`，一个结构体`TestChild`。接口里有一个方法`call`传入string

第一个匿名函数里我们为`TestChild`结构体添加了方法，这个方法继承了接口Test的`call`函数

最后实例化结构体，调用这个接口实现`call`方法