---
title: jquery获取checkbox数据
name: jquery-checkbox
date: 2019-09-02
id: 0
tags: [jquery,html]
categories: []
abstract: ""
---


使用**Jquery**获取复选框的数据


<!--more-->


使用**Jquery**获取复选框的数据

<!--more-->

当遇到批量操作的时候，会使用到复选框，使用原生js操作复选框有些复杂，使用jQuery简化了步骤

### 遍历复选框

```javascript
var chk_value =[];//定义一个数组    
$('input[name="png"]:checked').each(function(){//遍历每一个名字为png的复选框
	chk_value.push($(this).val());//将选中的值添加到数组chk_value中    
});
```

使用jQuery的`each`方法遍历每一个选中的复选框，并且获得里面的值

### 判断是否选中

```javascript
<input type="checkbox" id="test" class="test">
$('#test').is(":checked"); //选中返回true
$("#test").prop("checked")
```

### 添加选中属性

```javascript
$("#test").prop("checked","true")
$("#test").attr("checked","true")
```

### 移除选中属性

```javascript
$("#test").prop("checked","false")
$("#test").attr("checked","false")
```

