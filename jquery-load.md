---
title: jquery局部刷新页面
name: jquery-load
date: 2019-09-02
id: 0
tags: [jquery,html]
categories: []
abstract: ""
---


使用jQuery局部刷新页面的某个div块

<!--more-->

使用Jquery的`load`方法

```html
<div id="content">
          <div class="text">
           </div>
</div>
```

使用ajax发送数据

```javascript
$.ajax({
    async: false,
    type: "POST",
    url: '/',
    data: {"ids": str},
    dataType: "json",
    error: function () { },
    success: function () {
        $("#content").load(location.href+" .text");
        //window.location.reload();
    }
});
```

`load()`里的url即需要刷新的块，**注意**`.text`和页面地址间有空格

设置在ajax数据发送成功后，`sucess`函数执行，刷新页面

### 传统方式全局刷新

使用`window.location.reload();`可以达到全局页面刷新的目的