# 本地构建和预览临时说明

这个文件是临时操作说明，用来记录如何在本机预览当前 Jekyll/GitHub Pages 博客。

## 1. 进入项目目录

```sh
cd /home/jetio/Documents/repo/repo_extra/xrp001.github.io
```

## 2. 安装依赖

第一次运行，或者 `Gemfile.lock` 有变化后，执行：

```sh
bundle install
```

如果依赖已经安装过，可以跳过这一步。

## 3. 只构建检查

如果只是检查能不能正常生成静态网站，执行：

```sh
bundle exec jekyll build
```

构建成功后，会生成 `_site/` 目录。

## 4. 启动本地预览服务

执行：

```sh
bundle exec jekyll serve --host 127.0.0.1 --port 4000
```

看到类似下面的信息，说明启动成功：

```text
Server address: http://127.0.0.1:4000
Server running... press ctrl-c to stop.
```

## 5. 打开本地网站

在浏览器打开：

```text
http://127.0.0.1:4000
```

常用页面：

```text
http://127.0.0.1:4000/archives/
http://127.0.0.1:4000/categories/
http://127.0.0.1:4000/tags/
http://127.0.0.1:4000/about/
```

## 6. 修改后怎么预览

本地服务运行时，修改文章、模板或配置后，Jekyll 通常会自动重新生成页面。

如果浏览器没变化，刷新页面即可。

如果修改了 `_config.yml` 后没有生效，建议先停止服务，再重新启动。

## 7. 停止本地预览服务

如果服务是在当前终端前台运行的，按：

```text
Ctrl + C
```

如果服务是后台运行的，可以先查找进程：

```sh
ps aux | grep 'jekyll serve'
```

然后根据输出里的进程号停止它：

```sh
kill 进程号
```

例如进程号是 `12345`，就执行：

```sh
kill 12345
```

如果是 Claude Code 帮你启动的后台服务，可以让 Claude 停止对应后台任务。
