---
layout: post
title: "Docker + ttyd + tmux + Caddy 搭建低资源占用、跨平台兼容的 Web Terminal"
date: 2026-06-03
categories: tutorial
tags: [Docker, Linux, 终端, ttyd, tmux, Caddy, 教程]
---

在运维和远程开发场景中，经常需要随时随地通过浏览器访问 Linux 终端，手机、平板、电脑都能用，关闭浏览器后再次打开还能继续之前的终端状态，同时支持长期运行 Claude Code、Codex、SSH 会话，资源占用尽可能低，不依赖 VNC 和桌面环境。经过一系列踩坑和优化，最终采用 Docker + ttyd + tmux + Caddy 的组合方案，内存占用仅 10~30MB，CPU 基本为 0。

## 整体架构

```
浏览器
  ↓
Caddy（认证）
  ↓
ttyd（Web Terminal）
  ↓
tmux（会话保持）
  ↓
Linux Shell
```

最终效果：支持 macOS、Linux、Windows 浏览器，支持手机访问，支持中文显示，支持 Claude Code，浏览器关闭后会话保持。

## 为什么选择 ttyd

ttyd 是一个非常轻量级的 Web Terminal，相比 VNC、Guacamole、Xfce Desktop、Windows Remote Desktop，优势在于：

- 资源占用极低
- 部署简单
- 启动速度快
- 支持 WebSocket
- 支持移动端

## 为什么需要 tmux

仅使用 ttyd 时，如果执行 `vim test.txt` 后关闭浏览器，再次打开就会丢失编辑状态，重新进入 Shell。加入 tmux 后，关闭浏览器几小时再打开，仍停留在 vim 编辑界面。对于 Claude Code、SSH、长时间脚本、编译任务尤其重要。

## 为什么不使用 ttyd 自带认证

最开始的配置：

```bash
ttyd -c username:password
```

电脑浏览器正常，但手机访问却经常出现 `Press Enter to reconnect`，服务端日志显示 `User code denied connection`。表现为桌面 Chrome 正常、Linux Chrome 正常、Android Chrome 偶发失败、Safari 兼容性较差、微信内置浏览器异常。最终确认问题出在 ttyd 自带认证机制上。

改为由 Caddy 处理 Basic Auth 后彻底解决：

```
浏览器 → Caddy Basic Auth → ttyd（无认证）→ tmux → Shell
```

## 为什么选择 Caddy

相比 Nginx，Caddy 配置更简单，自带 Basic Auth，自动 HTTPS，自动申请 Let's Encrypt 证书，配置量极少。认证配置：

```caddy
:7681 {
    basicauth {
        admin <密码哈希>
    }

    reverse_proxy ttyd:7681
}
```

即可完成认证。

## 部署目录

```
/opt/ttyd/
├── docker-compose.yml
├── Dockerfile
├── start.sh
├── Caddyfile
└── data/
```

## Dockerfile

```dockerfile
FROM tsl0922/ttyd:alpine

RUN apk add --no-cache \
    tmux \
    bash \
    curl \
    vim \
    openssh-client \
    fontconfig

ENV HOME=/data
ENV TMUX_TMPDIR=/data/.tmux

RUN mkdir -p /data/.tmux /data/.local /data/.config

WORKDIR /data
```

基于 ttyd 的 Alpine 镜像，额外安装 tmux、bash、curl、vim、openssh-client、fontconfig 等常用工具。

## start.sh

```sh
#!/bin/sh
set -e

mkdir -p /data/.tmux /data/.local /data/.config
chmod 700 /data/.tmux

export HOME=/data
export TMUX_TMPDIR=/data/.tmux
export SHELL=/bin/bash
export TERM=xterm-256color
export COLORTERM=truecolor

cat > /data/.tmux.conf <<'EOF'
set -g status off
setw -g aggressive-resize on
set -g mouse on
set -g history-limit 10000

set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",xterm-256color:Tc"
set -as terminal-features ",xterm-256color:RGB"
set -as terminal-features ",tmux-256color:RGB"
EOF

tmux has-session -t main 2>/dev/null || tmux new-session -d -s main

exec ttyd \
  -p 7681 \
  -W \
  -m 0 \
  -P 30 \
  -T xterm-256color \
  -t rendererType=canvas \
  -t fontFamily="Menlo,Monaco,Consolas,monospace" \
  -t fontSize=13 \
  -t lineHeight=1.1 \
  -t letterSpacing=-0.3 \
  -t scrollback=10000 \
  -t disableLeaveAlert=true \
  tmux new-session -A -D -s main
```

启动脚本中关键点：写入 `tmux.conf` 配置以启用真彩色和鼠标支持，使用 `aggressive-resize on` 让 Pane 自动适配浏览器窗口尺寸，最后用 `tmux new-session -A -D -s main` 重建会话。

## docker-compose.yml

```yaml
version: "3.8"

services:
  ttyd:
    build:
      context: .
      dockerfile: Dockerfile

    image: local/ttyd-tmux:latest
    container_name: ttyd
    restart: unless-stopped

    expose:
      - "7681"

    volumes:
      - ./data:/data
      - ./start.sh:/usr/local/bin/start.sh:ro

    entrypoint:
      - /bin/sh
      - /usr/local/bin/start.sh

    mem_limit: 256m
    cpus: 0.5

  caddy:
    image: caddy:2-alpine
    container_name: ttyd-caddy
    restart: unless-stopped

    depends_on:
      - ttyd

    ports:
      - "7681:7681"

    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
```

通过 `mem_limit` 和 `cpus` 限制容器资源上限，即使长期运行也不会占用过多服务器资源。

## Caddyfile

首先生成密码哈希：

```bash
docker run --rm caddy:2-alpine \
  caddy hash-password \
  --plaintext '你的密码'
```

然后写入：

```caddy
:7681 {
    basicauth {
        admin <密码哈希>
    }

    reverse_proxy ttyd:7681
}
```

## 构建与启动

```bash
cd /opt/ttyd

chmod +x start.sh

docker compose build

docker compose up -d
```

查看日志：

```bash
docker logs -f ttyd
```

正常输出 `Listening on port: 7681` 即表示启动成功。

## 访问

浏览器打开 `http://服务器IP:7681`，输入用户名 `admin` 和配置的密码即可进入终端。

## 踩坑记录

### 1. ttyd 自带认证导致手机无法访问

现象：手机浏览器显示 `Press Enter to reconnect`，服务端日志 `User code denied connection`。解决：移除 ttyd 认证，改用 Caddy Basic Auth。

### 2. 中文全部显示为下划线

原因是字体配置问题。解决：使用浏览器本地字体，不要强制指定 CJK 字体。

### 3. Claude Code 底部内容被裁切

tmux 会话尺寸未正确同步。使用 `tmux new-session -A -D -s main` 重建会话，其中 `-D` 会断开旧客户端，按当前浏览器尺寸重新建立会话。

### 4. macOS 字体间距过大

最初尝试 JetBrains Mono、Noto Sans CJK、DejaVu Sans Mono 效果都不理想。最终发现 Menlo、Monaco、Consolas 由浏览器自行选择本地字体效果最佳：

```bash
-t fontFamily="Menlo,Monaco,Consolas,monospace"
-t fontSize=13
-t letterSpacing=-0.3
```

可同时兼容 macOS Chrome 和 Linux Chrome。

### 5. Linux 与 macOS 字体表现不一致

`letterSpacing` 调整过度会导致 macOS 好看但 Linux 挤在一起。最终折中值 `letterSpacing=-0.3` 两边效果均可接受。

## 资源占用

实际测试 CPU 约 0%，内存 10~30MB。即使长期运行 Claude Code、SSH、Shell、Python 脚本，资源占用依然非常低。

## 总结

```
浏览器
  ↓
Caddy
  ↓
ttyd
  ↓
tmux
  ↓
Shell
```

整套方案具备浏览器访问、手机访问、会话保持、Claude Code 支持、中文支持、跨平台字体兼容、Docker Compose 一键部署、HTTPS 扩展能力。对于个人服务器、开发环境、远程 AI Coding 环境来说，是一个简单、稳定、资源占用极低的 Web Terminal 方案。
