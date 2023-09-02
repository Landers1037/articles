---
title: 使用JS实现表单传输
name: sb6
date: 2019-08-27
id: 0
tags: [js]
categories: []
abstract: ""
---


不在html创建表单，使用js动态创建传输post数据

<!--more-->

### html结构

```html
<button type="button" onclick="delete_img()">删除</button>
```

### javascript创建空表单

```javascript
function delete_img(PARAMTERS) {
        var temp_form = document.createElement("form");
            temp_form.action = '/url/';
            //如需打开新窗口，form的target属性要设置为'_blank'
            temp_form.target = "_self";
            temp_form.method = "post";
            temp_form.style.display = "none";
            //添加参数
         for (var item in PARAMTERS) { //建立表单
               var opt = document.createElement("textarea");
               opt.name = PARAMTERS[item].name;
               opt.value = PARAMTERS[item].value;
               temp_form.appendChild(opt);
            }
            document.body.appendChild(temp_form);
            //提交数据
            temp_form.submit();
    }
```

最终将函数`delete_img()`绑定到button上，点击后就会发送post数据

### js创建有效表单

```javascript
function input() {
	var form = document.createElement('form');
    form.action = '/index/';
    form.method = 'post';      
    var input = document.createElement('input');
    input.type = 'hidden';
    input.name = 'key'; //input的name属性
    input.value = "test"; //input默认值
    form.appendChild(input);   
    $(document.body).append(form);
    form.submit();
}
```

