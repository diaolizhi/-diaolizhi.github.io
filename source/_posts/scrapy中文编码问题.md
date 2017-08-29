---
title: scrapy 中文编码问题
date: 2017-8-30
categories: python
---

深夜踩坑...
<!--more-->
# 问题描述

python 版本: 2.7

scrapy 版本: 1.4

scrapy 用保存文件的时候再打开总是类似于
```
\u9999\u6e2f\uff08\u7e41\u4f53\uff09
```
刚开始是复制到在线解析的网站上面,后来不满足于此,尝试谷歌解决,经过将近两个小时的搜索才解决问题...

# 解决方案
在 settings.py 文件下添加一句:
```python
FEED_EXPORT_ENCODING = 'gbk'
```

# 总结
一开始以为 encode 一下就可以了,弄了半天并没有什么卵用.搜索到的也基本没有用,可能是我想的关键字不对,后来看到[一篇博客](https://caiknife.github.io/blog/2013/08/02/scrapy-json-solution/)(居然是 2013 年的),这里说的方法我看不懂,后来在 Stack Overflow 看到
>Since Scrapy 1.2.0, a new setting FEED_EXPORT_ENCODING is introduced. By specifying it as utf-8, JSON output will not be escaped.

>That is to add in your settings.py:

>FEED_EXPORT_ENCODING = 'utf-8'

说实话当时我试了没用,也没有意识到为什么不对,没错我根本不理解这几种编码,最后看到网上有人说用记事本转换成 ANSI 编码,马上想到 settings.py 里面的这句话,完全是运气...

网上搜索出的还是 0.24 的版本,现在都 1.4 了,英语不好看不懂官方文档真的不行.
