---
title: hexo主题配置
name: hexo-theme1
date: 2019-02-06
id: 0
tags: [hexo]
categories: []
abstract: ""
---


# 感谢主题的设计者

针对主题的模板文件修改，找到layout下的ejs文件

网页左侧的信息栏	`base-profile.ejs`
网页底部标题	`base-footer.ejs`
网页侧边栏	  `base-sidebar.ejs`

<!--more-->


# 感谢主题的设计者

针对主题的模板文件修改，找到layout下的ejs文件

网页左侧的信息栏	`base-profile.ejs`
网页底部标题	`base-footer.ejs`
网页侧边栏	  `base-sidebar.ejs`
<!--more-->

## 在信息栏添加自己喜欢的文字

```javascript
<% if(theme.school.enable) { %>
        <div class="school">
            <a href = "<%- url_for("http://www.hust.edu.cn/") %>" >&nbsp;&emsp;&emsp;&emsp;&thinsp;HUST 1037</a>
        </div>
    <% } %>
```

然后只需要在主题配置文件_config.yml中添加

```javascript
school:
  enable: true

```

## 在css文件中修改文字样式

找到`D:\Github Page\themes\archer\src\scss\_partial\_partial\profile.scss`
添加内容

```scss
.school {
  padding: 0.5rem 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  a {
    font-size: 1.2rem;
    font-weight: bold;
  }
}
```

## 空格修改网页元素的位置

### html常用的空格样式

```html
&nbsp; #牛逼空格是最常用的空格
&ensp; #嗯空格是1/2的中文字符长度
&emsp; #恶魔空格是1格中文字符长度
&thinsp; #瘦子空格是一个很窄的空格
#记住不要忘记分号
```

### html常用的分割线

```html
<hr> # 一条分割线
<HR style="FILTER: alpha(opacity=100,finishopacity=0,style=3)" width="80%" color=#987cb9 SIZE=3> # 一条颜色渐变的分割线
<HR style="FILTER: alpha(opacity=100,finishopacity=0,style=1)" width="80%" color=#987cb9 SIZE=3> # 一边颜色渐变的分割线
```

## 添加人数统计

旋转地球统计 [官网](https://www.revolvermaps.com/)

快速插入js代码

```javascript
<script type="text/javascript" src="//ra.revolvermaps.com/0/0/6.js?i=0n29o7blvb7&amp;m=7&amp;c=e63100&amp;cr1=ffffff&amp;f=arial&amp;l=0&amp;bv=90&amp;lx=-420&amp;ly=420&amp;hi=20&amp;he=7&amp;hc=a8ddff&amp;rs=80" async="async"></script>
```

## 压缩hexo的css和html文件

安装插件 `all_minifier`
修改hexo目录下的配置文件，添加`all_minifier: true`

## 禁止git自动转义LF到CRLF

```bash
git config --global core.autocrlf flase
```

