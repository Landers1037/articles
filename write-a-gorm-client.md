---
title: 写一个gorm client
name: write-a-gorm-client
date: 2020-11-04
id: 0
tags: [go]
categories: []
abstract: ""
---


使用`gorm`库写一个调用数据库类

<!--more-->

使用`gorm`可以方便地对数据库进行orm映射操作

我们写一个简易的类封装对gorm的使用

### 定义类

定义一个结构体`MyOrm`

```go
import "github.com/jinzhu/gorm"

type MyOrm struct {
    DB *gorm.DB
}
```

在结构体中定义一个属性`DB`，类型为gorm提供的数据库类型

接下来就可以调用`MyOrm.DB`来使用gorm的操作

### 封装方法

为了方便使用我们可以自己封装函数

封装连接函数

```go
func (d *MyOrm) Connect(db string) *gorm.DB {
	dbc, err := gorm.Open("sqlite3", db)
	if err != nil {
		return dbc
	}
	d.DB = dbc
	// 每次数据库使用完毕应该关闭
	return dbc
}
```

封装Find函数

```go
// 通过结构体查询
func (d *MyOrm) FindByStruct(condition interface{}, table interface{}) *gorm.DB {
	return d.DB.Where(condition).Find(table)
}

func (d *MyOrm) FindAll(table interface{}, where ...interface{}) *gorm.DB {
	return d.DB.Find(table, where...)
}
```

> 注意：使用`args...`语法糖定义参数数组时，在传入时需要通过`...args`方式取出内容

此时我们可以新建一个**MyOrm**类

```go
var user User

myOrm := MyOrm{}
myOrm.Connect("test.db") // 连接数据库
myorm.FindAll(&user, User{Name: "bell"})
```

根据结构体`User{}`传值查询

### 原生方法

通过调用`MyOrm.DB.xxx`可以使用gorm的原生方法

例如

```go
myOrm := MyOrm{}
myOrm.Connect("test.db") // 连接数据库
myorm.DB.Find(&user, User{Name: "bell"})
```

### 注意

每次操作完毕数据库需要关闭数据库连接

```go
func (d *MyOrm) Close() *gorm.DB {
	_ = d.DB.Close()
	return d.DB
}
```

使用`defer`保证数据库的关闭

可能出现的空指针错误

```go
myOrm := MyOrm{}
myOrm.Connect("test.db") // 连接数据库
myorm.DB.Find(&user, User{Name: "bell"})
myorm.Close()
//
myorm.DB.Find(&user, User{Name: "nana"})
```

当执行第二次查询时就会出现空指针错误，因为在此之前已经调用了Close关闭了数据库连接

同样的每次使用前应该先连接数据库调用`Connect()`函数否则也会出现数据库空指针错误

### 表名定义

gorm默认使用复数表名

当结构体名称为`User`时，默认的表名是全小写的`users`复数形式

这就导致了结构体命名和默认表名的不一致性，可以通过设置`*DB.SingularTable(true)`保证表名始终为单数

或者使用`TableName`结构体函数覆盖默认属性

```go
type User struct {}

func (User) TableName() string {
    return "user"
}
```

返回一个自定义的`string`类型表名