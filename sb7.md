---
title: 随笔-Flask用户登录的完整实现
name: sb7
date: 2019-08-28
id: 0
tags: [flask]
categories: []
abstract: ""
---


记使用Flask-login实现完整的用户登录流程

<!--more-->

### 安装依赖

```bash
$ pip3 install flask-login
```

### 用户的初始化

```python
from flask_login import LoginManager, current_user, UserMixin, login_required, login_user, logout_user
login = LoginManager(app)
```

```python
@login.user_loader
def load_user(user_id):  # 创建用户加载回调函数，接受用户 ID 作为参数
    if user_id == users['id']:
        user = User()
        user.id = user_id
        return user  # 返回用户对象

login.login_view = 'login'
class User(UserMixin):
    pass
```

`id`只是用来识别用户的，简单起见直接使用用户名也可以。演示`users`里只有一个用户，它可以是一个用户列表

`login.login_view = 'login'`作为默认用户登录视图，所有登录都会路由到`/login`

`User`类直接继承**UserMixin**类简化用户创建的流程

### 用户信息定义

```python
users =  {'id':'root', 'username': 'root', 'password': '123456'}
#简单起见使用字典直接保存用户信息，正常情况应该使用数据库
```

### 登录视图

```python
@app.route('/login',methods=['GET','POST'])
def login():
    next = '' #登陆后的跳转页面凭据
    if request.args.get("next"):
        next = request.args.get("next")

    if request.method == 'POST' and len(next)>0:
        getus = request.form["login[username]"]
        getpw = request.form["login[password]"]
        if getus == users["username"] and getpw == users["password"]:
            curr_user = User()
            curr_user.id = getus
            # 通过Flask-Login的login_user方法登录用户
            login_user(curr_user)

            return redirect(next) #重定向到验证后的页面
        else:
            return redirect(url_for('login')) #如果登录失败就重定向到登录页面

    return render_template('login.html')
```

### 需要验证登录的路由

```python
@app.route('/test/')
@login_required
def test():

    return render_template('test.html')
```

使用`@login_required`装饰器修饰路由，在进入前需要登录验证

在未验证时返回的是即将跳转的路由，如http://127.0.0.1:5000/?next=/test其中的`next`键对应的值就是需要跳转的页面地址，所以在`/login`路由下有一句判断，获取需要跳转的地址，然后使用`redirect`在登录成功后重定向

```python
if request.args.get("next"):
	next = request.args.get("next")
```

#### login页面

创建简单的登录表单

```html
<form class="form-horizontal" method="post">
	<div class="heading">Login</div>
	<div class="form-group">
		<i class="fa fa-user"></i><input required name="login[username]" type="text" autocomplete="off" class="form-control" placeholder="Username" id="exampleInputEmail1">
	</div>
	<div class="form-group">
		<i class="fa fa-lock"></i><input required name="login[password]" type="password" class="form-control" placeholder="Password" />
	</div>
	<div class="form-group">
		<button type="submit" class="btn btn-default"><i class="fa fa-arrow-right"></i></button>
		<span>Don't have an account? <a href="#" class="create_account">Sign up</a></span>
	</div>
</form>
```

用户名`login[username]`，密码`login[password]`，使用`.form[key]`的方式获取表单数据

```python
getus = request.form["login[username]"]
getpw = request.form["login[password]"]
```

#### 用户验证

```python
#从表单获取数据后和本地存储的用户信息对比
if getus == users["username"] and getpw == users["password"]:
	curr_user = User()
	curr_user.id = getus #这里因为id和用户名相同所以省略了id获取uesrname的判断
	# 通过Flask-Login的login_user方法登录用户
	login_user(curr_user)
```

至此就简单实现了用户登录的流程

### 用户登出

用户登出需要创建登出的视图`/logout`

```python
@app.route('/logout')
@login_required
def logout():
    logout_user() #调用登出方法
    return render_template('logout.html')
```

因为实际的登出并不需要一个页面，所以我们可以在登出完成后重定向回到主页

```python
return redirect(url_for('index'))
```

### 页面保护需求

如果只是对某个页面需要登录验证，并且其页面数据只是部分需求验证，即对于访客隐藏部分数据

可以在页面中添加**验证**

```python
#使用装饰器是最简单的验证方案
@login_required

#还可以使用登录重定向的方式
from flask_login import login_required, current_user

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        if not current_user.is_authenticated:  # 如果当前用户未认证
            return redirect(url_for('index'))  # 重定向到主页
        # ...
```

使用Jinjia2模板对页面部分数据加密

```html
{% if current_user.is_authenticated %}
<div>
    <p>
        我是加密内容
    </p>
</div>
{% endif %}
```

在用户未登录时，加密的内容就不会被显示出来

### 总结

以上便是flask登录的全部流程

- 初始化app`login = LoginManager(app)`
- 创建用户数据库
- 创建用户登录视图`/login`，用户登出视图`/logout`
- 初始化用户`User`类，用户加载函数`load_users`
- 在需要登录验证的视图后添加装饰器`@login_required`
- 创建登录页面表单
- 如需页面内容保护，导入`current_user`类