---
title: 在Notepad++中实现Markdown代码高亮以及使用插件实现可视化
date: 2018-12-08
categories: tutorial
tags: [markdown,notepad++,教程]
---
## Notepad++

选择一个强大而好用的文本编辑器，是进行Web开发和编程必不可少的一部分，甚至对于通常的写作，一个舒服的文本编辑器也会让你写起文字来觉得优雅而潇洒。Sublime Text是一款不错的编辑器，简洁且跨平台，但对新手来说配置起来有些麻烦，对于通常使用Windows的用户来说，Notepad++或许是一个更好的选择。Notepad++（NPP），顾名思义，就是一个加强版的记事本了，虽然只多了两个加号，但功能甩系统记事本可不是一里两里。

![notepad++](http://wx3.sinaimg.cn/large/6a8c0fe1gy1fxz52mmmb7j205o05o0sm.jpg)

Notepad++本身除了作为一款强大的编辑器外，主要魅力还有由NPP社区和开发人员贡献的种类众多的插件，这些插件几乎可以满足我们各种各样的需求，解决大大小小的问题。找到适合自己的插件，并且灵活使用，将有效提高我们的开发和写作效率。

## Markdown代码高亮

### 下载所需文件

#### 下载链接

[代码库](https://github.com/xrp001/markdown-plus-plus)

[点此直接下载文件压缩包](https://codeload.github.com/xrp001/markdown-plus-plus/zip/master)
### 导入语法规则
打开Notepad++，点击“语言” ，选择“自定义语言格式” ，点击“导入”，选择下载并解压后文件夹"theme-default"中的“userDefineLang_markdown.xml”文件安装默认的主题，也可以是其它文件夹下的主题。

导入完成后重启notepad++，点击“语言”，选择“Markdown”即可。
![](http://wx2.sinaimg.cn/large/6a8c0fe1gy1fxz6iw9tn3j20ng0hct98.jpg)
## 安装MarkdownViewer++插件实现实时可视化
在Plugin Admin下安装MarkdownViewer++：

依次打开 插件(P)-->Plugin Admin。

在Available下寻找MarkdownViewer++插件后，点击install：
![](http://wx3.sinaimg.cn/large/6a8c0fe1gy1fxz519lsmij20l00edq3a.jpg)

安装完成后应重启Notepad++即可。
![](http://wx4.sinaimg.cn/large/6a8c0fe1gy1fxz51a1z4lj21g10t7wju.jpg)

参考：<br />
[
在Notepad++中进行Markdown文档的视图可视化](https://blog.csdn.net/weixin_42192321/article/details/81076017)<br />
[Notepad++中实现Markdown语法高亮与实时预览](http://www.cnblogs.com/wupeixuan/p/8673362.html)<br />
