---
title: Hello World! 我的第一篇文章
date: 2026-07-25
updated: 2026-08-02
---

历尽千辛万苦终于搭好了博客，得开始写一点东西了。

打算先在第一篇文章中介绍一下这个博客是如何搭建的~

其实可以直接对着 Hexo 官方文档搭建，但官方文档有点简略了。。因此还是写成完整一点的小教程吧，也是备忘。

## 基本规划

调研了一下，为了省去大部分麻烦（其实是前端忘光了），我选用 Hexo 来搭建静态博客，然后将其部署到 Github Pages 上，其中可以借助 Github Actions 来完成自动化部署。

## Hexo

首先自行安装 Node.js，然后 npm 全局安装 Hexo。

```bash
$ npm install -g hexo-cli
```

接着找一个目录 `<folder>` 作为博客项目的根目录，初始化博客项目。

```bash
# 方法 1：原地初始化
$ mkdir <folder>
$ cd <folder>
$ hexo init
# 方法 2：创建目录并初始化
$ hexo init <folder>
$ cd <folder>
```

安装一下相关依赖。

```bash
$ npm install
```

生成博客文件，可以本地运行查看效果。

```bash
$ hexo g
$ hexo s
```

## Git & Github

首先自行安装 Git，然后在根目录中初始化本地仓库。

```bash
$ cd <folder>
$ git init
```

注意，按照官方文档，需要检查一点：默认情况下 `public/` 不会被上传(也不该被上传)，确保 `.gitignore` 文件中包含一行 `public/`。

先进行第一次 commit ~

```bash
$ git add .
$ git commit -m "init"
```

在 Github 上创建一个远程仓库，名为 `<username>.github.io`，其中 `<username>` 为 Github 用户名。

回到本地目录，推送。

```bash
# 将当前分支强制命名为 master
$ git branch -M main
# 关联刚刚创建的远程仓库
$ git remote add origin https://github.com/<username>/<username>.github.io.git
# 推送并设置上游跟踪
$ git push -u origin main
```

这样，远程仓库上就有了我们的博客项目代码。

## Github Pages & Github Actions

### 杂谈

据我的了解，Github Pages & Github Actions 的作用大致如下：

- Github Pages 可以帮我们托管静态博客（静态网站都行），类似的服务还有 Vercel、Cloudflare Pages 等。最重要的是，能够提供一个免费域名`<username>.github.io`（以前我弄过一个域名，要备案还是太麻烦了）。
- Github Actions 的作用就是可以自动化我们的工作流，就比如 `pages.yml` 的作用大概是当我们把博客项目代码 push 到仓库的时候 Github Actions 就会自动给我们部署（部署到 Github Pages 上）。

这两个东西感觉都挺有用的，但我平常实在是太懒了，一直都是听说过但没用过。。以后折腾下。

### 实操

首先记录一下 Node.js 版本。

```bash
$ node --version
```

这里记作 `<node_version>`。

接着，进入 Github 远程仓库，点击 Settings > Pages > Source，将 source 更改为 GitHub Actions。这样，Github Pages 和 Github Actions 就关联起来了。

回到本地仓库，创建 Github Actions 目录和文件 `.github/workflows/pages.yml`。

内容如下，其中 `node-version: "<node_version>"` 的值是刚刚记录的 Node.js 版本。

```yml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "<node_version>"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

接着我们依旧提交并推送到远程仓库

```bash
$ git add .
$ git commit -m "use Github Actions"
$ git push
```

稍等片刻，等待 Github Actions 部署完后，我们访问 `<username>.github.io` 就能看到博客了。

当然后续还有很多美化需要配置。。我之前配置 [NexT](https://github.com/next-theme/hexo-theme-next) 主题就费了好些功夫，还是需要多看文档多学呀~

## 后续计划

后续可能会随缘更新一下文章，当然大部分是流水账，主要还是给自己看的，哈哈哈~