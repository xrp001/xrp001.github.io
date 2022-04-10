---
layout: post
title: "Mac无法访问github解决办法和简单工具分享"
date: 2022-04-10
categories: tutorial
tags: [教程]

---

最近刚上手MacBook写代码，在提交代码到github时，发现会因为网络问题提交失败。

解决办法如下：

1、在<https://www.ipaddress.com>获取以下地址的IP：

 ```bash

  github.com
  github.global.ssl.fastly.net
  assets-cdn.github.com
 ```

 简单写了个Python代码自动获取以上地址的IP：

 ```python
 import re
 import requests
 from bs4 import BeautifulSoup
 
 websites = ["github.com",
             "github.global.ssl.fastly.net",
             "assets-cdn.github.com"
             ]
 
 # 执行所有的HTML页面的解析
 all_datas = []
 htmls = []
 for website in websites:
     url = f"https://ipaddress.com/website/%s" % website  # 页面链接
     print("craw html:", url)
     r = requests.get(url)
 
     soup = BeautifulSoup(r.text, 'html.parser')
     ip_info = (
         soup.find_all("tbody", id="dnsinfo")
     )
     info = ip_info[0].find_all("a")
     for i in info:
         ip = i.get_text()
         if re.search(r'[a-z]', ip) != None:
             break
         all_datas.append(ip+" " + website)
 
 print(all_datas)
 
 with open("hosts.txt", "w") as fout:
     for data in all_datas:
         fout.writelines(data + "\n")
 ```

2、进入mac终端，执行vi /etc/hosts，输入密码后，按i进入编辑，在最后加入以下内容：

 ```bash
 140.82.113.4 github.com
 199.232.69.194 github.global.ssl.fastly.net
 185.199.108.153 assets-cdn.github.com
 185.199.109.153 assets-cdn.github.com
 185.199.110.153 assets-cdn.github.com
 185.199.111.153 assets-cdn.github.com
 ```

  以上内容加入后，按esc退出编辑，并输入`:wq`进行保存并退出
