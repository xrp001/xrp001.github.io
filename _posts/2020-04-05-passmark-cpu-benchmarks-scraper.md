---
title: 通过TXT列表在PassMark里获取CPU Benchmarks
date: 2020-04-05
categories: tutorial
tags: [scraper,juypter notebook,教程]
---

# Passmark CPU Benchmarks Scraper
Scrape CPU benchmarks from PassMark by .txt list.

Note: The function is imperfect, very simple.


## Usage

 For single CPU:
Modify the code in ```CPU_Info-single.ipynb``` and use it to scrape the information of a single CPU. The scraped information will be saved in ```CPU_Info-single.json```.

For multiple CPU:
First specify the CPU list that needs to be scraped in ```cpu_list.txt```, and then use ```CPU_Info-multiple.ipynb``` to scrape. The scraped information will be saved in ```CPU_Info-multiple.json```.

![cpu_list.txt](https://wx4.sinaimg.cn/large/6a8c0fe1gy1gdj6ubw93aj206z0h5753.jpg)

Note that while you can do whatever you care to with this code, the data you scrape with it is for your personal use only.
http://www.passmark.com/legal/disclaimer.htm
