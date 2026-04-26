---
layout: post
title: "GNOME 顶栏系统监视扩展 System Monitor Beautiful 分享"
date: 2026-04-27
categories: tutorial
tags: [Linux, Ubuntu, GNOME, 工具]
---

最近给 Ubuntu 22.04 写了一个轻量级的 GNOME Shell 顶栏系统监视扩展，在顶栏直接显示 CPU、内存、网络速率，不需要打开系统监视器就能实时看到系统状态。

项目地址：<https://github.com/xrp001/system-monitor>

## 主要功能

✅ 顶栏实时显示：
- CPU 使用率百分比
- 内存使用率百分比
- 网络下载 / 上传实时速率

✅ 自动选择默认路由网卡，找不到时自动回退到最活跃网卡

✅ 可自定义刷新间隔：1 / 2 / 3 / 5 / 10 秒可选

✅ 下拉菜单支持查看进程排行：
- CPU 占用最高的进程
- 内存占用最高的进程
- 网络流量最高的进程
- 显示进程名 + PID，数量可在设置里调整 1~10 个

✅ 基于原生 `/proc` 接口采样，低开销设计，适合后台常驻运行

---

## 安装方法

### 一键安装（推荐）

> 注意：请以普通用户身份运行，**不要加 sudo**，脚本会在需要的时候自动请求权限。

```bash
git clone https://github.com/xrp001/system-monitor.git
cd system-monitor
./install.sh
```

安装完成后按 `Alt + F2`，输入 `r` 回车重启 GNOME Shell 即可生效。

---

## 卸载方法

```bash
./uninstall.sh
```

同样重启 GNOME Shell 就完全卸载了。

---

## 性能开销说明

这个扩展在设计时就特别注意了资源占用问题，在有 600+ 进程的机器上测试结果：

| 使用模式 | CPU 额外开销 |
|---------|-------------|
| 关闭进程排行 | 几乎可以忽略 |
| 开启进程排行 + 3秒刷新 | 约 1~2% 单核 |
| 开启进程排行 + 5秒刷新 | < 1% 单核 |

常驻内存增量只有 1~2 MB 左右，日常使用完全不会感觉到卡顿。

---

## 常见问题

### 安装后顶栏什么都不显示？

1. 先确认扩展已经启用：
```bash
gnome-extensions info system-monitor-beautiful@$USER
```

2. 查看日志排查问题：
```bash
journalctl /usr/bin/gnome-shell -f
```

### Wayland 会话怎么重启？

Wayland 下无法热重启 GNOME Shell，只能注销后重新登录。

可以用这个命令查看当前会话类型：
```bash
echo $XDG_SESSION_TYPE
```

### 可以用 root 安装吗？

不行，GNOME 扩展必须安装在当前用户目录下，用 sudo 安装会装到 root 用户目录里，普通用户看不到。

---

这个扩展目前适配的是 Ubuntu 22.04 / GNOME 42 版本，后续会慢慢支持更新的版本。有问题可以在 GitHub 提 Issue。