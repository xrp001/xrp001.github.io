---
title: Jetson TX2/Xavier 刷机资料及注意事项
date: 2018-12-08
categories: tutorial
description: Jetson TX2/Xavier 刷机资料简单汇总
tags: [xavier,tx2,jetson,flash,刷机,教程]
---
## 刷机前注意事项
1. 刷机需要使用一台安装 Ubuntu 或者虚拟机安装 Ubuntu 系统的主机（HOST），在 Ubuntu 系统内安装对应版本的 JetPack 对 TX2 或者 Xavier 进行刷机。
2. 在 NVIDIA 官网下载 JetPack 时**需要账号**，可以在下载时也可以提前注册好（**可能需要连接代理翻墙，方法之一：[shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5) [中文文档](https://github.com/shadowsocks/shadowsocks-qt5/wiki)**）。
3. 下载 JetPack 可能也需要代理。
4. 在使用JetPack的安装过程中会**大概率**遇到下载失败问题，若重试多次扔无法解决，请使用代理。

## Jetson TX2 刷机
__官方英文版教程__：<br />
* [JetPack with Linux for Tegra® Documentation Rev.3.3.180716](https://docs.nvidia.com/jetson/archives/jetpack-archived/jetpack-33/index.html#jetpack/3.3/install.htm%3FTocPath%3D_____3)


__非官方中文参考教程__：<br />
* [Xavier (2)——刷机](https://blog.csdn.net/haoqimao_hard/article/details/83447696)<br />
* [NVIDIA Jetson Xavier通过JetPack 4.1刷机教程（虚拟机版）](https://blog.csdn.net/x111y1j1/article/details/83269507)<br />

## Jetson AGX Xavier 刷机
__官方英文版教程__：<br />
* [JetPack with Linux for Tegra® Documentation Rev.4.1.1.181105](https://docs.nvidia.com/jetson/archives/jetpack-archived/jetpack-411/index.html#jetpack/4.1.1/install.htm%3FTocPath%3D_____3)

__非官方中文参考教程__：<br />
* [02-NVIDIA Jetson TX2 通过JetPack 3.1刷机完整版（踩坑版）](https://www.jianshu.com/p/bb4587014349)<br />

## 其它资源下载
[Jetson Download Center](https://developer.nvidia.com/embedded/downloads)

NVIDIA Jetson 下载中心，可以下载 TX2/Xavier 专用的 Tensorflow 、JetPack 等软件以及开发板的各种数据资料。

**最新的官方英文版教程**可以在最新版本的 JetPack 下载页面获取，以 JetPack 4.1.1 DP 为例：

![](http://wx1.sinaimg.cn/large/6a8c0fe1gy1fxzbo9qz62j211y0dt0sw.jpg)
