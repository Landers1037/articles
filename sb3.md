---
title: 随笔-移动端适配
name: sb3
date: 2019-08-21
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


本来以为移动端的适配是件不简单的事情，闲着无聊看了下原生js实现的移动端跳转，看着不是很难就顺手对mgek页面首页做了移动端优化

<!--more-->

### 移动端判断

首先需要使用js对设备进行判断，在`head`头部加入js代码

```javascript
<script>
if(/Android|webOS|iPhone|iPod|BlackBerry|IEMobile|OperaMini/i.test(navigator.userAgent
		{window.location = "/mobile/";}
</script>
```

因为ipad的屏幕尺寸已经足够作为pc端判定了，就把ipad剔除了
判断设备为移动设备后网页就会自动跳转到`'/mobile/'`专为移动设备优化的页面了

### 移动端优化

因为移动端的屏幕太小，mgek的部分图片是作为`background-image`显示的，为了图像保持纵横比就会被压缩的很小，因为是针对移动设备的页面就没必要管它在大屏pc端的显示效果了

#### 调整排列

```html
<div class="row">
    <div class="col-md-6 col-md-offset-3 slider-text"></div>
</div>
```

原本为`col-md-6`每行两个的布局，在移动端统一设置为`col-md-12`,`col-sm-12`让图片单张占据全部宽度

#### 调整间隔

为了显示更加贴合移动设备，修改了一下padding

```css
.container-wrap{
	margin-bottom: 0;
}
#fh5co-hero .flexslider .slider-text > .slider-text-inner .btn{
     width: 50%;
     background-color: white;
     border: 1.5px solid #222222;
     border-radius: 5px;
     padding: 0px 5px 0px 5px!important;
}
```

