---
title: Scrapy中的css选择器的使用
name: css1
date: 2019-03-02
id: 0
tags: [python,scrapy]
categories: []
abstract: ""
---


## Scrapy提供两种选择器分析网页的源代码

## xpath和css

因为css的高效简单，我大部分时间使用css分析简单的网页结构，xpath适合分析网页结构更为复杂的页面

<!--more-->


## Scrapy提供两种选择器分析网页的源代码

## xpath和css

因为css的高效简单，我大部分时间使用css分析简单的网页结构，xpath适合分析网页结构更为复杂的页面
<!--more-->

## 使用方法

`.css('选择器语句').extract()`

注意extract提取的返回是列表，extract_first提取的是其中的第一项

## css选择器

|                |                 |                                |
| -------------- | --------------- | ------------------------------ |
| 选择器         | 实例            | 解释                           |
| .class         | .name           | 选择类名name的节点后的所有元素 |
| #id            | #name           | 选择id为name的节点后的所有内容 |
| .class a       | .name a         | 选择class为name后的所有a节点   |
| a div          | a div           | 选择a节点后的所有div节点       |
| a[title]       | a[title]        | 选择属性含有title的所有a节点   |
| a::text        | a::text         | 提取a节点内的文字              |
| a::attr(href)  | a::attr(href)   | 提取a节点内的href属性的文字    |
| :nth-child(n)  | a:nth-child(2)  | 提取第二个a节点                |
| :nth-child(2n) | a:nth-child(2n) | 提取2的倍数的a节点             |
| *              | a*              | 提取a后面的所有元素            |

