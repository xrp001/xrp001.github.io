---
layout: post
title: "基于 Docker 部署 Mihomo + MetaCubeXD 统一认证代理服务"
date: 2026-06-13
categories: tutorial
tags: [Docker, Linux, Mihomo, MetaCubeXD, 代理, 教程]
---

## 适用场景

本文介绍如何基于 Docker 部署一套支持认证、规则切换和统一出口的代理服务。

适用于：

- Ubuntu 22.04 / 20.04
- Debian 11/12
- CentOS 7
- Rocky Linux 8/9
- AlmaLinux 8/9
- OpenEuler
- 其他支持 Docker 的 Linux 发行版

本文以 Ubuntu Server 22.04 为例，其它 Linux 发行版仅需根据自身环境调整 Docker 与防火墙安装方式即可。

## 功能目标

- Docker 容器化部署
- 支持 Clash/Mihomo 订阅
- 支持 HTTP/SOCKS5 代理
- 支持用户名密码认证
- 支持 Rule / Global / Direct 模式切换
- 支持 Web UI 管理
- 支持多个客户端统一接入
- 支持 Docker 容器使用
- 支持 Claude Code、OpenWebUI、CC-Switch 等工具使用
- 配置持久化
- 订阅自动更新
- 节点状态实时查看
- 流量与连接实时监控

---

## 架构

```text
                    ┌─────────────────┐
                    │  MetaCubeXD UI  │
                    │   Web 管理界面   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     Mihomo      │
                    │   代理核心引擎   │
                    └────────┬────────┘
                             │
                  HTTP/SOCKS5 认证代理
                             │
      ┌──────────────┬──────────────┬──────────────┐
      │              │              │
      ▼              ▼              ▼

 Linux主机      Docker容器      Claude Code
 OpenWebUI      CC-Switch      其他客户端
```

---

## 创建目录

```bash
mkdir -p proxy-stack/mihomo && \
touch proxy-stack/docker-compose.yml proxy-stack/mihomo/config.yaml && \
chown -R $(id -u):$(id -g) proxy-stack
```

目录结构：

```text
proxy-stack/
├── docker-compose.yml
└── mihomo/
    └── config.yaml
```

---

## docker-compose.yml

```yaml
services:

  mihomo:
    image: metacubex/mihomo:latest
    container_name: mihomo

    restart: unless-stopped

    ports:
      - "7890:7890"
      - "9090:9090"

    volumes:
      - ./mihomo:/root/.config/mihomo

    cap_add:
      - NET_ADMIN

    devices:
      - /dev/net/tun:/dev/net/tun

    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "5"

  metacubexd:
    image: ghcr.io/metacubex/metacubexd:latest

    container_name: metacubexd

    restart: unless-stopped

    ports:
      - "9999:80"
```

---

## mihomo/config.yaml

```yaml
mixed-port: 7890

allow-lan: true

bind-address: "*"

mode: rule

log-level: info

ipv6: false

external-controller: 0.0.0.0:9090

secret: 你的管理密钥

authentication:
  - "用户名:密码"

profile:
  store-selected: true
  store-fake-ip: true

dns:
  enable: true
  ipv6: false
  listen: 0.0.0.0:1053

  enhanced-mode: fake-ip

  nameserver:
    - 223.5.5.5
    - 119.29.29.29

  fallback:
    - https://dns.google/dns-query
    - https://dns.cloudflare.com/dns-query

proxy-providers:

  airport:
    type: http
    url: "你的订阅地址"
    path: ./providers/airport.yaml
    interval: 3600

proxy-groups:

  - name: PROXY
    type: select
    use:
      - airport

rules:
  - MATCH,PROXY
```

说明：

- `secret` 是 Mihomo 外部控制 API 的管理密钥，用于 MetaCubeXD 等 Web UI 连接 `9090` 管理接口。
- `authentication` 是 HTTP/SOCKS5 代理连接认证，用于客户端访问 `7890` 代理端口。
- 两者用途不同，建议分别设置不同的强密码。

---

## 启动服务

```bash
cd proxy-stack

docker compose pull

docker compose up -d
```

检查：

```bash
docker ps
```

---

## 配置 MetaCubeXD

访问：

```text
http://服务器IP:9999
```

填写：

```text
API:
http://服务器IP:9090

Secret:
你的管理密钥
```

连接成功后即可：

- 查看节点
- 切换节点
- 查看连接
- 查看流量
- 查看日志
- 切换代理模式

---

## 代理模式

### Rule

根据规则自动决定是否走代理。

### Global

所有流量全部走代理。

### Direct

所有流量全部直连。

推荐：

```text
生产环境：Rule
测试环境：Global
```

---

## Linux 主机使用代理

HTTP：

```bash
export HTTP_PROXY=http://用户名:密码@服务器IP:7890
export HTTPS_PROXY=http://用户名:密码@服务器IP:7890
```

SOCKS5：

```bash
export ALL_PROXY=socks5://用户名:密码@服务器IP:7890
```

永久生效：

```bash
echo 'export HTTP_PROXY=http://用户名:密码@服务器IP:7890' >> ~/.bashrc
echo 'export HTTPS_PROXY=http://用户名:密码@服务器IP:7890' >> ~/.bashrc
source ~/.bashrc
```

---

## Docker 容器使用代理

```yaml
services:

  app:

    image: ubuntu

    environment:

      HTTP_PROXY: http://用户名:密码@服务器IP:7890

      HTTPS_PROXY: http://用户名:密码@服务器IP:7890

      ALL_PROXY: socks5://用户名:密码@服务器IP:7890
```

---

## Claude Code 使用代理

```bash
export HTTP_PROXY=http://用户名:密码@服务器IP:7890
export HTTPS_PROXY=http://用户名:密码@服务器IP:7890
```

验证：

```bash
env | grep PROXY
```

---

## OpenWebUI 使用代理

```yaml
environment:

  HTTP_PROXY: http://用户名:密码@服务器IP:7890

  HTTPS_PROXY: http://用户名:密码@服务器IP:7890
```

---

## 验证代理

查看出口 IP：

```bash
curl -x http://用户名:密码@服务器IP:7890 https://api.ipify.org
```

查看出口地区：

```bash
curl -x http://用户名:密码@服务器IP:7890 https://ifconfig.co/json
```

访问 Google：

```bash
curl -L \
-x http://用户名:密码@服务器IP:7890 \
https://google.com
```

认证失败验证：

```bash
curl -x http://错误用户:错误密码@服务器IP:7890 https://google.com
```

应返回：

```text
407 Proxy Authentication Required
```

---

## 日志查看

实时日志：

```bash
docker logs -f mihomo
```

最近100行：

```bash
docker logs --tail 100 mihomo
```

---

## 配置持久化

由于已经挂载：

```yaml
volumes:
  - ./mihomo:/root/.config/mihomo
```

以下内容会持久化：

```text
config.yaml
providers/*
cache.db
```

包括：

- 节点订阅
- 节点选择
- Fake-IP缓存
- 部分运行状态

---

## 防火墙配置

推荐开放：

| 端口 | 用途 | 访问范围 |
|------|------|------|
| 7890 | HTTP/SOCKS5代理 | 客户端 |
| 9999 | MetaCubeXD | 管理网 |
| 9090 | Mihomo API | 管理网 |

UFW 示例：

```bash
ufw allow 7890/tcp

ufw allow from 192.168.16.0/24 to any port 9999 proto tcp

ufw allow from 192.168.16.0/24 to any port 9090 proto tcp
```

---

## 常见问题

### 301 Moved

```bash
curl https://google.com
```

返回 301 属于 Google 正常跳转。

解决：

```bash
curl -L https://google.com
```

### 429 Rate Limit

访问 ipinfo.io 返回 429。

说明代理正常，只是目标网站限流。

### 407 Proxy Authentication Required

检查：

```yaml
authentication:
  - "用户名:密码"
```

### unexpected eof

通常为节点异常或节点失效。

处理方式：

- 切换节点
- 切换 Global 模式测试
- 检查订阅是否正常

---

## 最终效果

```text
MetaCubeXD:
http://服务器IP:9999

Mihomo API:
http://服务器IP:9090

代理地址:
http://用户名:密码@服务器IP:7890
```

即可为 Linux 主机、Docker 容器、Claude Code、OpenWebUI 等服务统一提供带认证的代理访问能力。
