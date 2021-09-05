---
title: sqlalchemy使用
name: sqlalchemy
date: 2020-03-08
id: 0
tags: [sql,python]
categories: []
abstract: ""
---


使用sqlalchemy的ORM来操作数据库


<!--more-->


使用sqlalchemy的ORM来操作数据库

<!--more-->

安装sqlalchemy

```python
pip install sqlalchemy
```

### 映射关系

#### 数据库

数据库中的表，数据库，字段在python中全部映射为类来处理

`Engine` 连接数据库，在python中作为连接数据库的桥梁

`Session`连接池，事务，用作进行数据库的chax

`Model`表，对应python中的类

`Column`列，字段，对应python类的属性

`Query`若干行，用于查询

`sqlachemy`对常用的数据库的字段进行了封装

| 数据类型  | 对应的数据库类型 | python中        |
| --------- | ---------------- | --------------- |
| Integer   | int              | int             |
| String    | varchar          | string          |
| Text      | text             | string 长字符串 |
| Float     | float            | float           |
| Boolean   | tinyint          | bool            |
| Date      | date             | datetime        |
| Time      | time             | datetime,time   |
| Date Time | datetime         | datetime,time   |

#### 数据库的连接

```python
from sqlalchemy create_engine

engine = create_engine('sqlite:///app.db') #数据库引擎
```

初始化一个数据库连接，`engine`内部维护了一个pool连接池和dialect方言

可选的参数

`echo=True	`	是否打印出执行的sql语句

`pool_size=5` 	连接池的大小

`pool_recycle=60*30`	限制数据库超时的时间

#### 表的创建初始化

此时我们创建一个User表

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Table,Column,Integer,String,Text,Boolean

base = declarative_base()
class User(base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True)
    name = Column(String,nullable=False)
    age = Column(Integer,nullable=False)
```

`base`通过内置的方法`declarative_base()`创建，是一个基类用于和数据库表进行关联

 `__tablename__` 是表名，通过该表明python可以把这个类和数据库表关联

`Column`即字段，我们定义了三个字段id，name，age。其中`id`字段设置为主键，python会自动把它设置为自增的形式作为唯一的主键

#### 生成数据库表

使用`base.metadata.create_all(engine)`即可生成表，传入的参数即刚才连接的数据库对象

### 操作数据库

`session`被用于创建python和数据库之间的会话，由它操作数据库

```python
from sqlalchemy.orm import sessionmaker

Session= sessionmaker(bind=engine) #session池
session = Session() #一个seesion用于操作数据库
```

使用`sessionmaker(bind=engine)`函数创建一个工厂用于关联engine，保证每一个session的处理都会和engine关联

#### 增

```python
u = User(name="bi",age=2)
session.add(u)
session.commit()

###添加多条
session.add_all([
       Users(name='qiaozhi'),
       Users(name='peiqi')
])
```

实例化一个`User`类，然后使用`add`添加到session维护的空间里，使用`commit`来提交改变

#### 查

```python
session.query(User).first() #取出第一个
session.query(User).filter_by(name="b").all() #取出名称为b的全部对象
session.query(Users).filter(Users.id == 4)
```

过滤分为`filter`和`filter_by`，其中`fliter_by`仅支持=,<,>,!=运算符

此外**filter**还支持使用`and`,`or`,`in`等sql判断

#### 改

```python
session.query(User).first().update({'name':'bb'})
###
user = session.query(User).fiter_by(name="b").first()
user.name= "bi"
session.add(user)
session.commit()
```

有两种更新的方式，直接在query中使用`update`方法，或者通过操作模型类

#### 删

```python
session.query(User).first().delete()
###
user = session.query(User).fiter_by(name="b").first()
user.name= "bi"
session.delete(user)
session.commit()
```

删除也支持两种方式，直接使用`delete`方法或通过操作模型类

**注意同直接使用sql相同，最后的sql事务只有提交才会作用到数据库**

#### 排序

```python
session.query(Users).order_by(Users.name.desc()).all() #按照名称的顺序
session.query(Users).order_by(Users.name.desc(), Users.id.asc()).all()
```

### 外键的使用

```python
from sqlalchemy.orm import relationship,backref
from sqlalchemy import ForeignKey
    
class User(base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True)
    name = Column(String,nullable=False)
    age = Column(Integer,nullable=False)
    
class Book(base):
    __tablename__ = 'book'
    id = Column(Integer,primary_key=True)
    name = Column(String)
    user_id = Column(Integer, ForeignKey('user.id') 
    user = relationship("user", backref="book")
```

在这里`user_id`外键，一个user可以对应多个book

`user`外键第一个参数是正向引用，即book引用user(这本书对应的用户是谁)

第二个参数是反向引用，即user引用book(这个用户拥有的书)

所以在查询时

```python
a = session.query(Book).filter_by(name="book1").first()
a.user.name

###
b = session.query(User).filter_by(name="peiqi").first()
b.book
```

第一个在书籍中查询book1的拥有者，第二个在用户中查找peiqi拥有的书

#### 约束关系

外键有四种约束关系

| 关系      | 表示                             |
| --------- | -------------------------------- |
| RESTRICT  | 父表数据删除后，子表拒绝删除数据 |
| NO ACTION | 不采取动作                       |
| CASCADE   | 父表数据删除，子表同步删除       |
| SET NULL  | 父表数据删除，子表置null         |

