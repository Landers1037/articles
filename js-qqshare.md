---
title: js添加分享按钮
name: js-qqshare
date: 2019-08-07
id: 0
tags: [html]
categories: []
abstract: ""
---


使用原生js和jQuery为网页添加qq分享按钮

调用了官方的分享api


<!--more-->


使用原生js和jQuery为网页添加qq分享按钮

调用了官方的分享api

<!--more-->

```javascript
<script>
         function qqFriend() {
	var p = {
	    /*获取URL，可加上来自分享到QQ标识，方便统计*/
	    url: 'http://www.xxx.com',
	    desc: '',
	    /*分享标题(可选)*/
	    title: '个人主页',
	    /*分享摘要(可选)*/
	    summary: '一个神奇的地方',
	    /*分享图片(可选)*/
	    pics: '',
	    /*视频地址(可选)*/
	    flash: '',
	   /*分享来源(可选) 如：QQ分享*/
	   site: '',
	   style: '201',
	   width: 32,
	   height: 32
};
        var s = [];
	for(var i in p) {
	   s.push(i + '=' + encodeURIComponent(p[i] || ''));
	}
	var url = "http://connect.qq.com/widget/shareqq/index.html?" + s.join('&');
	window.open(url);
}
</script>
```

#### 使用jQuery和share.js

引入jQuery.js和social-share.js

```html
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
<script src="https://cdn.bootcss.com/social-share.js/1.0.16/js/social-share.min.js"></script>
<link href="https://cdn.bootcss.com/social-share.js/1.0.16/css/share.min.css" rel="stylesheet">
```

新建div容器初始化

```html
<div class="social-share"></div>
```

默认类为social-share时不需要添加配置

如果需要禁用部分按钮，使用`data-disabled`参数

```html
<div class="social-share" data-disabled="google,diandian,douban,wechat,linkedin"></div>
```

更多参见官方文档和源码

[share.js](https://www.oschina.net/p/share-js)

