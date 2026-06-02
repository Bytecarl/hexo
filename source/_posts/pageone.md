# 搭建教程

Markdown

```
---
title: 从零开始：使用 Hexo + GitHub + Cloudflare Pages 搭建极速免费个人博客
date: 2026-06-02 21:05:00
tags:
  - Hexo
  - Cloudflare
  - 博客搭建
categories:
  - 技术分享
---

这是一篇真正面向国内网络环境的 Hexo 博客搭建与云端部署终极通关指南。本文将完整记录如何绕过 Git 网络超时、多层目录嵌套崩溃、以及主题配置全白等一系列实战暗坑。

## 🛠 一、 本地环境准备与破局初始化

在 Windows 环境下，传统的初始化命令经常会因为 GitHub 直连网络崩溃而中断。我们推荐使用干净的子目录配合国内镜像源来一气呵成搞定。

### 1. 规范创建本地独立目录
打开终端（CMD），切入你想存放博客的根目录，创建一个完全干净的新文件夹：
```bash
mkdir myblog
cd myblog
```

### 2. 使用 Gitee 镜像加速初始化

为了防止 `hexo init` 因为网络超时而卡死崩溃，强制使用国内镜像拉取核心框架：

Bash

```
hexo init --clone [https://gitee.com/hexojs/hexo-starter.git](https://gitee.com/hexojs/hexo-starter.git)
```

*注：即使中途因默认主题下载触发 GitHub 报错，只要最后显示 `INFO Start blogging with Hexo!` 即代表基础框架已安全拷贝成功。*

### 3. 配置国内 npm 镜像并补全依赖

Bash

```
npm config set registry [https://registry.npmmirror.com](https://registry.npmmirror.com)
npm install
```

## 🚀 二、 核心配置与 Butterfly 主题排坑

在配置主题（如大热的 `Butterfly`）时，有两大致命语法和逻辑暗坑必须避开。

### 1. 严格遵守 YAML 语法规则（冒号加空格）

打开 `_config.butterfly.yml` 修改导航栏菜单（`menu`）时，**任何冒号 `:` 后面如果还有内容，必须紧跟一个【空格】**，否则会导致本地编译彻底全白！

YAML

```
menu:
  首页: / || fas fa-home
  时间轴: /archives/ || fas fa-archive
  标签: /tags/ || fas fa-tags
```

### 2. 初始化核心路由页面

光在配置文件里写上路径还不够，必须在本地终端真正把这些页面“生”出来：

Bash

```
hexo new page tags
hexo new page categories
```

建好后，务必打开对应的 `index.md`，在 Front-matter 区域内加上对应的类型声明（注意冒号后有空格）：

YAML

```
type: "tags"
```

### 3. 升级 Hexo 核心与主题兼容

如果本地执行 `hexo generate` 时因为旧主题不兼容新版 Hexo 变量而报 `TypeError`，直接利用 npm 强行升级生效的主题包即可：

Bash

```
npm install hexo-theme-butterfly@latest --save
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

修复后运行 `hexo clean && hexo generate`，看到 `INFO xx files generated` 即代表本地编译大获全胜。

## 📦 三、 源码同步至 GitHub

### 1. 为 Git 命令行套上代理

国内主动 Push 代码到 GitHub 必须通过本地代理软件。查看你软件的“混合监听端口”（例如 `10808` 或 `7890`），并在终端强制 Git 走代理：

Bash

```
git config --global http.proxy [http://127.0.0.1:10808](http://127.0.0.1:10808)
git config --global https.proxy [http://127.0.0.1:10808](http://127.0.0.1:10808)
```

### 2. 规范推送到远程仓库

在 GitHub 上创建一个名为 `hexo` 的公共（Public）空仓库，随后在本地依次执行：

Bash

```
git init
git remote add origin [https://github.com/你的用户名/hexo.git](https://github.com/你的用户名/hexo.git)
git branch -M main
git add .
git commit -m "feat: 完美初始化博客源码"
git push -u origin main
```

## ☁️ 四、 托管至 Cloudflare Pages（大招通关）

Cloudflare Pages 是目前托管静态博客最完美的免费方案，自带全球 CDN 且支持全自动 CI/CD 构建。

### 🚨 避坑核心：不要错选成 Worker！

在 Cloudflare 控制台，务必进入 **“Workers 和 Pages” -> 点击“创建” -> 切换到“Pages”标签页**，然后点击 **“连接到 Git”** 并选中你的仓库。

### 🔧 黄金参数对号入座配置：

由于我们本地代码存放在 `myblog` 子目录下，在 Pages 的构建配置页必须严格按照以下参数填写：

- **项目名称**：`你的博客名字`（决定你的免费二级域名前缀）
- **生产分支**：`main`
- **框架预设 (Framework preset)**：选择 **`Custom`（自定义）** 或 `Hexo`
- **根目录 (Root directory)**：⚠️ **输入 `myblog`**（告诉系统钻进子文件夹去找代码，否则会报 Root directory not found）
- **构建命令 (Build command)**：输入 **`npx hexo generate`**
- **构建输出目录 (Build output directory)**：⚠️ **输入 `public`**（极其关键！这是 Hexo 编译后生成的纯静态网页夹）

点击 **“保存并部署”**，静候一分钟，看到所有对勾亮起，你的个人博客就正式在互联网上通过专属域名（或你绑定的自定义域名）完美上线了！

## ✍️ 五、 以后写博客的日常三部曲

这套工作流最爽的一点就是支持“自动化构建”。以后你写了新文章，本地不需要跑编译和部署，只需要执行雷打不动的 Git 提交三部曲：

Bash

```
git add .
git commit -m "发布了一篇新文章"
git push -u origin main
```

GitHub 收到代码后会立刻通知 Cloudflare，云端服务器会自动在几秒钟内完成编译并更新你的网站。

祝你的独立博客之旅一路顺风！

```
---

### 💡 建议
把这段内容保存到 `myblog/source/_posts/how-to-build-hexo.md` 里，然后你在本地运行：
```bash
hexo clean && hexo generate
```

紧接着用我们的“日常更新三部曲”推送到 GitHub，几秒钟后，刷新你的域名 `bytecarl.de`，这篇完美的通关教程就会作为你的第一篇大作，精美地展现在所有人面前啦！