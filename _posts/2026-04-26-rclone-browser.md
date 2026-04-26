---
layout: post
title: "Rclone Browser - 最好用的 rclone 图形化客户端"
date: 2026-04-26
categories: tutorial
tags: [工具, Linux, Windows, macOS, 云存储]
---

一直在找好用的 rclone 图形界面，最近发现了这个不错的开源客户端 Rclone Browser，原生跨平台支持 Windows、macOS、Linux，而且已经完整汉化了。

项目地址：<https://github.com/xrp001/RcloneBrowser>

## 为什么需要这个工具

rclone 本身是非常强大的命令行云存储同步工具，支持几乎所有主流网盘，但是命令行操作对于日常管理文件还是很不方便。

Rclone Browser 做了非常友好的图形封装，不需要学习复杂的命令参数，所有功能都可以用鼠标点几下完成。

---

## 主要功能

✅ 完全兼容原生 rclone 配置，直接读取你已经配置好的所有网盘
✅ 多标签页同时打开多个远程存储，切换方便
✅ 完整的文件管理：上传下载、新建文件夹、重命名、删除
✅ 后台队列任务，支持同时多个传输作业，不卡界面
✅ 支持直接拖拽上传，从本地文件管理器拖进去就可以
✅ 可以直接挂载远程存储到本地磁盘
✅ 支持流媒体直接播放，调用系统播放器打开远程视频
✅ 生成网盘文件公开分享链接
✅ 任务模板可以保存重复使用的传输任务
✅ 原生深色主题支持
✅ ✅ **完整中文界面**

---

## 下载安装

在 GitHub Releases 页面可以直接下载各平台预编译版本：

<https://github.com/xrp001/RcloneBrowser/releases>

- Windows：安装包，支持 Win7 及以上
- macOS：dmg 镜像
- Linux：AppImage 格式，全发行版通用，还支持树莓派 ARM 架构

> 💡 Linux 用户推荐安装 AppImageLauncher 获得更好的桌面集成体验。

---

## 使用说明

1.  首先你需要先安装好 rclone 本身并配置好你的网盘
2.  打开 Rclone Browser 它会自动识别你的 rclone.conf 配置
3.  左侧会列出你所有已经配置的远程存储，双击就可以打开
4.  所有文件操作和本地文件管理器用法一样

这个工具不会修改你的任何 rclone 配置，你随时可以换回命令行使用。

---

## 编译安装

如果需要自己编译也很简单：

### Ubuntu / Debian
```bash
sudo apt install build-essential cmake qtbase5-dev qttools5-dev
git clone https://github.com/xrp001/RcloneBrowser.git
cd RcloneBrowser
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### macOS
```bash
brew install qt5 cmake
git clone https://github.com/xrp001/RcloneBrowser.git
cd RcloneBrowser
mkdir build && cd build
cmake ..
make -j$(sysctl -n hw.ncpu)
make dmg
```

---

目前这是我找到的维护最活跃、功能最完整的 rclone 图形客户端，而且没有任何广告和多余的东西，完全开源免费。