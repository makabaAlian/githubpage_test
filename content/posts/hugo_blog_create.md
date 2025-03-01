+++
date = '2025-02-26T22:29:47+08:00'
draft = false
title = 'Hugo + github pages 建站'
+++

# hugo 建站流程
> 很久以前就用hugo搭过一个blog，挂在自己的ECS上，但是当时没有坚持写博文，后面ECS没续费了，也就渐渐忘了这事了。现在把它重新搭建一下，记录一下建站的流程，这篇文章也算是久违的回归了。

## 安装环境配置
- 系统：`Windows 11`
- 终端：`git bash`

## 安装hugo
1. 这里选择预编译好的Windows版本包下载: [hugo_extended_0.144.2_windows-amd64.zip](https://github.com/gohugoio/hugo/releases/download/v0.144.2/hugo_extended_0.144.2_windows-amd64.zip)
2. 下载完成后，解压到任意目录，不要有中文路径，比如`D:/hugo`
3. 添加到系统环境变量PATH中
4. 打开终端，输入`hugo version`命令，查看hugo是否安装成功

有疑问可以看一下官方的安装文档，各种安装姿势都说明了，这里推荐`extended`版本的hugo，某些主题需要`extended`版本才能正常运行。
> [https://gohugo.io/installation/windows/#prebuilt-binaries](https://gohugo.io/installation/windows/#prebuilt-binaries)

## 新建站点
```shell
# 任意位置打开终端，输入hugo new site xxxx命令，xxxx是你的站点名。
hugo new site xxxx
# 切换到站点目录
cd xxxx
# 初始化git仓库
git init
# 新建主题, 如果clone不下来，使用国内镜像地址: gitclone.com 添加到 github.com之前
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
# 更改配置文件hugo.toml，记得改一下theme路径 themesDir = "themes" 默认配置跑不起来
cp themes/LoveIt/exampleSite/hugo.toml hugo.toml
# 启动本地服务器, 浏览器输入localhost:1313 就可以看到本地站点了
hugo server
```
## 创建文章
```shell
# 切换到站点目录
cd xxxx
# 创建文章
hugo new posts/my-first-post.md
# 编辑文章，注意格式，yaml头部必须有title和date字段，draft字段为true表示草稿，false表示发布
# 正常使用 hugo server 看不到草稿文章，需要加参数 -D
# 最新版的hugo server已经会默认自动热更新了，编辑文章保存后浏览器自动刷新，不需要重启hugo server
hugo server -D
```
## 部署到github pages
> github pages 是github提供的静态页面托管服务，我们可以直接把site推到github上，配置一个github action就可以自动部署发布到github pages；这里我们假设你已经有github账号了。
>>新建一个仓库，比如`your-username.github.io`，注意仓库名必须是`your-username.github.io`，这样github才会自动识别为你的站点仓库,；还需要对仓库进行一些设置，具体可以看官方文档，这里不再赘述, 链接: [https://gohugo.io/hosting-and-deployment/hosting-on-github/](https://gohugo.io/hosting-and-deployment/hosting-on-github/)。
```shell
# 克隆刚才的站点仓库到本地
git clone https://github.com/your-username/your-username.github.io.git
# 切换到站点目录
cd your-username.github.io
# 复制刚才的站点目录下的所有文件到本仓库
cp -r ../xxxx/* .
# 添加workflow脚本
mkdir -p .github/workflows
touch .gihub/workflows/hugo.yaml
```
`hugo.yaml`内容如下, 拷贝自官方文档，支持的hugo版本为`0.144.2`：
```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.141.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
```shell
# 提交到github
git add .
git commit -m "first commit"
git push
```
等待github action部署完成，稍等片刻，就可以在`https://your-username.github.io`看到你的站点了。

## 尾注
- 折腾了两晚上，看了不少pages服务商，后面试试接入到腾讯的Edge One上，趁着免费赶紧嫖一波
- github action这有个坑，如果你的子模块用镜像站拉下来的，action在checkout时可能会卡住，记得改回去

