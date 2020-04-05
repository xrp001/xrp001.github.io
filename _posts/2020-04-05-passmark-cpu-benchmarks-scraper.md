---
title: 通过TXT列表在PassMark里抓取CPU Benchmarks
date: 2020-04-05
categories: tutorial
tags: [scraper,juypter notebook,教程]
---

> 最近在考虑入手软路由，可是可选择的CPU太多，性能跑分和功耗千差万别，很难进行对比，本来自己做了个表格用于对比，可是最近[PassMark](www.cpubenchmark.net)里的数据在最近进行了更新，很多CPU的跑分都变了，做的表格失效了，于是修改了别人的代码用来爬取Passmark上的CPU信息来作为参考。

# Passmark CPU Benchmarks Scraper
Scrape CPU benchmarks from [PassMark](www.cpubenchmark.net) by .txt list.

通过TXT列表从[PassMark](www.cpubenchmark.net)中抓取CPU Benchmarks。

Note: The function is imperfect, very simple.

注意：此功能不完善，过于简陋。

Code repository link: [https://github.com/xrp001/passmark-cpu-benchmarks-scraper.git](https://github.com/xrp001/passmark-cpu-benchmarks-scraper.git)

代码仓库地址：[https://github.com/xrp001/passmark-cpu-benchmarks-scraper.git](https://github.com/xrp001/passmark-cpu-benchmarks-scraper.git)

## Usage

## 食用方法

**For single CPU**:

**对于单个CPU**：

Modify the code in ```CPU_Info-single.ipynb``` and use it to scrape the information of a single CPU. The scraped information will be saved in ```CPU_Info-single.json```.

修改```CPU_Info-single.ipynb```中的代码并且用其来抓取单个CPU的信息，抓取到的信息将被存放到文件```CPU_Info-single.json```中。

**For multiple CPU**:

**对于多个CPU**：

First specify the CPU list that needs to be scraped in ```cpu_list.txt```, and then use ```CPU_Info-multiple.ipynb``` to scrape. The scraped information will be saved in ```CPU_Info-multiple.json```.

先在```cpu_list.txt```中指定需要抓取的CPU清单，然后使用```CPU_Info-multiple.ipynb```来抓取这些CPU的信息，抓取到的信息将被存放到文件```CPU_Info-single.multiple```中。

![cpu_list.txt](https://wx4.sinaimg.cn/large/6a8c0fe1gy1gdj6ubw93aj206z0h5753.jpg)

**Note that while you can do whatever you care to with these codes, the data you scrape with it is for your personal use only.**

**请注意，尽管您可以使用这些代码做任何您想做的事情，但您从中抓取的数据仅供您个人使用。**

[http://www.passmark.com/legal/disclaimer.htm](http://www.passmark.com/legal/disclaimer.htm)
