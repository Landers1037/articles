---
title: wordcloud制作词云
name: wordcloud
date: 2019-02-14
id: 0
tags: [wordcloud]
categories: []
abstract: ""
---


# 注意事项：

结巴库是中文分词，原理是把gbk编码转换为utf-8，所以导入的文本一定要是gbk编码，否则会出现编码错误<!--more-->

```python
import jieba
import wordcloud
import matplotlib.pyplot as plt

txt = open('./txt.txt', 'r').read()
words_ls = jieba.cut(txt, cut_all=True)
words_split = " ".join(words_ls)

wc = wordcloud.WordCloud(
    width=800,
    height=600,
    background_color="#ffffff",  # 设置背景颜色
    max_words=500,  # 词的最大数（默认为200）
    max_font_size=60,  # 最大字体尺寸
    min_font_size=10,  # 最小字体尺寸（默认为4）
    colormap='bone',  # string or matplotlib colormap, default="viridis"
    random_state=10,  # 设置有多少种随机生成状态，即有多少种配色方案
    mask=plt.imread("x.jpg"),  # 读取遮罩图片！！
    font_path='simhei.ttf'
)
my_wordcloud = wc.generate(words_split)

plt.imshow(my_wordcloud)
plt.axis("off")
plt.show()
# wc.to_file('zzz.png')  # 保存图片文件

```

