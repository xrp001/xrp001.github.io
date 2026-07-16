---
layout: post
title: "基于 Docker 部署 Nextcloud + Collabora 在线预览服务"
date: 2026-07-16
categories: tutorial
tags: [Docker, Nextcloud, Collabora, Linux, 在线预览, 教程]
---

本文介绍如何基于 Docker 部署 Nextcloud + Collabora，实现 DOC、XLS、PPT、PDF、视频等文件的在线查看与编辑，并通过外部存储挂载服务器上的本地文件。

> 本文使用 `192.168.10.100` 作为服务器示例 IP。部署时请替换为 Ubuntu 服务器的实际 IP，并为数据库设置独立的强密码。

## 1. 目录规划

```bash
mkdir -p /opt/share-files
mkdir -p /srv/share-files
cd /opt/share-files
```

本地文件放入：

```text
/srv/share-files
```

---

## 2. 完整 `docker-compose.yml`

```yaml
services:
  nextcloud:
    image: nextcloud:apache
    container_name: nextcloud
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      MYSQL_HOST: mariadb
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: "你的数据库密码"
    volumes:
      - ./nextcloud:/var/www/html
      - /srv/share-files:/external-files
    depends_on:
      - mariadb

  mariadb:
    image: mariadb:11.4
    container_name: nextcloud-mariadb
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED
    environment:
      MYSQL_ROOT_PASSWORD: "你的数据库Root密码"
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: "你的数据库密码"
    volumes:
      - ./mariadb:/var/lib/mysql

  collabora:
    image: collabora/code:latest
    container_name: nextcloud-collabora
    restart: unless-stopped
    ports:
      - "9980:9980"
    environment:
      aliasgroup1: "http://192\\.168\\.10\\.100:8080"
      aliasgroup2: "http://nextcloud:80"
      extra_params: >-
        --o:ssl.enable=false
        --o:ssl.termination=false
        --o:net.proto=IPv4
        --o:server_name=192.168.10.100:9980
    cap_add:
      - MKNOD
```

其中：

```text
192.168.10.100
```

是本文使用的示例 IP，请替换为 Ubuntu 服务器的实际 IP。`MYSQL_PASSWORD` 在 Nextcloud 和 MariaDB 服务中的值必须保持一致。

启动：

```bash
cd /opt/share-files
docker compose up -d
```

---

## 3. 初始化 Nextcloud

浏览器访问：

```text
http://192.168.10.100:8080
```

填写：

```text
管理员账号：自行设置
数据库用户：nextcloud
数据库密码：填写 docker-compose.yml 中设置的数据库密码
数据库名：nextcloud
数据库主机：mariadb
```

---

## 4. 配置可信域名

查看：

```bash
docker exec -u www-data nextcloud php occ config:system:get trusted_domains
```

设置：

```bash
docker exec -u www-data nextcloud php occ config:system:set trusted_domains 0 \
  --value="192.168.10.100:8080"

docker exec -u www-data nextcloud php occ config:system:set trusted_domains 1 \
  --value="nextcloud"
```

验证：

```bash
docker exec -u www-data nextcloud php occ config:system:get trusted_domains
```

预期：

```text
192.168.10.100:8080
nextcloud
```

---

## 5. 启用外部存储

启用应用：

```bash
docker exec -u www-data nextcloud php occ app:enable files_external
```

页面配置：

```text
管理设置
→ 外部存储
→ 新增本地存储
```

配置：

```text
名称：共享资料
类型：本地
路径：/external-files
```

通过“可用于”限制用户或用户组。

只读共享可将 Compose 改为：

```yaml
- /srv/share-files:/external-files:ro
```

---

## 6. 离线安装 `richdocuments`

下载兼容版本：

```bash
cd /opt/share-files

wget https://github.com/nextcloud-releases/richdocuments/releases/download/v11.0.1/richdocuments-v11.0.1.tar.gz
```

复制到容器：

```bash
docker cp richdocuments-v11.0.1.tar.gz nextcloud:/tmp/
```

解压：

```bash
docker exec -u root nextcloud bash -c '
cd /var/www/html/custom_apps &&
tar -xzf /tmp/richdocuments-v11.0.1.tar.gz &&
chown -R www-data:www-data richdocuments
'
```

启用：

```bash
docker exec -u www-data nextcloud php occ app:enable richdocuments
```

验证：

```bash
docker exec -u www-data nextcloud php occ app:list | grep richdocuments
```

---

## 7. 配置 Collabora 地址

设置内部访问地址：

```bash
docker exec -u www-data nextcloud php occ config:app:set richdocuments wopi_url \
  --value="http://nextcloud-collabora:9980"
```

设置浏览器访问地址：

```bash
docker exec -u www-data nextcloud php occ config:app:set richdocuments public_wopi_url \
  --value="http://192.168.10.100:9980"
```

激活配置：

```bash
docker exec -u www-data nextcloud php occ richdocuments:activate-config
```

正常应显示：

```text
Configured WOPI URL:
http://nextcloud-collabora:9980

Configured public WOPI URL:
http://192.168.10.100:9980
```

---

## 8. 验证服务链路

验证 Nextcloud 访问 Collabora：

```bash
docker exec nextcloud curl -s \
  http://nextcloud-collabora:9980/hosting/discovery | head
```

验证公开地址：

```bash
curl -s http://192.168.10.100:9980/hosting/discovery | head
```

验证 Collabora 访问 Nextcloud：

```bash
docker exec nextcloud-collabora bash -c '
exec 3<>/dev/tcp/nextcloud/80
printf "GET /status.php HTTP/1.1\r\nHost: nextcloud\r\nConnection: close\r\n\r\n" >&3
head -n 1 <&3
'
```

预期：

```text
HTTP/1.1 200 OK
```

验证 Discovery 公开地址：

```bash
docker exec nextcloud curl -s \
  http://nextcloud-collabora:9980/hosting/discovery |
grep -o 'urlsrc="[^"]*"' | head -1
```

预期包含：

```text
http://192.168.10.100:9980/browser/
```

---

## 9. 权限配置

创建只读用户：

```text
管理员头像
→ 用户
→ 创建用户
```

创建只读组，例如：

```text
viewer
```

在外部存储配置中，将“共享资料”仅授权给：

```text
viewer
```

底层强制只读：

```yaml
- /srv/share-files:/external-files:ro
```

重建：

```bash
docker compose up -d --force-recreate nextcloud
```

---

## 10. 支持的文件类型

| 文件类型 | 在线能力 |
|----------|----------|
| DOC/DOCX | Collabora 在线查看、编辑 |
| XLS/XLSX | 在线查看、编辑 |
| PPT/PPTX | 在线查看、编辑 |
| PDF | Nextcloud 内置预览 |
| MP4 | 浏览器在线播放 |
| Markdown | Nextcloud Text 应用预览 |
| HTML | 通常按文本查看或下载 |

---

## 11. 常用排查命令

查看 Collabora 日志：

```bash
docker logs --since 5m nextcloud-collabora 2>&1 | tail -n 300
```

查看 Nextcloud 日志：

```bash
docker exec nextcloud tail -n 200 /var/www/html/data/nextcloud.log
```

筛选 WOPI 错误：

```bash
docker logs --since 5m nextcloud-collabora 2>&1 | \
grep -Ei 'error|wopi|authorized|unauthorized|failed'
```

查看 Office 配置：

```bash
docker exec -u www-data nextcloud php occ config:list richdocuments
```

查看 Nextcloud 状态：

```bash
docker exec -u www-data nextcloud php occ status
```

---

## 12. 本次关键问题

### App Store 找不到应用

现象：

```text
Could not download app richdocuments
```

处理：

```text
手动下载 richdocuments 压缩包
→ 解压到 /var/www/html/custom_apps
→ app:enable richdocuments
```

### 浏览器无法打开 Collabora

原因：

```text
public_wopi_url 使用 Docker 内部域名 nextcloud-collabora
```

修复：

```text
内部地址：http://nextcloud-collabora:9980
公开地址：http://192.168.10.100:9980
```

### Trusted domain error

原因：

```text
nextcloud 未加入 trusted_domains
```

修复：

```bash
docker exec -u www-data nextcloud php occ config:system:set trusted_domains 1 \
  --value="nextcloud"
```

### No authorized hosts found

原因：

```text
Collabora 未授权实际 Nextcloud 地址
```

修复：

```yaml
aliasgroup1: "http://192\\.168\\.10\\.100:8080"
aliasgroup2: "http://nextcloud:80"
```
