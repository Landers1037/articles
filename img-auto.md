---
title: css设置图片宽度自适应
name: img-auto
date: 2019-07-05
id: 0
tags: [html]
categories: []
abstract: ""
---


有时网页需要针对不同宽度的设备优化结构，多张图片的位置不能固定，需要按照设备宽度自动排列


<!--more-->


有时网页需要针对不同宽度的设备优化结构，多张图片的位置不能固定，需要按照设备宽度自动排列

<!--more-->

假设网页结构

```html
    <div class="container">
        <td class="test"><img src="/static/alipay.png" style="max-width: 100%"/></td>
        <td class="test"><img src="/static/wechatpay.png" style="max-width: 100%" /></td>
    </div>
```

**重要**：即将图片的宽度设置为`max-width: 100%`

使用css设置容器的最大宽度为100%，即默认的设备的宽度，当设备宽度小于给定图片宽度时，图片就会重新排列

```css
.container{
    width: 100%;
    text-align: center;
}
.test{
    text-align: center;
    width: 100%;
    box-sizing: border-box;
    padding: 10px;
    min-width: 150px;
}
```

