# dihaifeng.github.io

## 我的公众号
![公众号](https://github.com/dihaifeng/dihaifeng.github.io/raw/main/static/images/qrcode_for_gh_027d39279844_430.jpg)

## Mac 环境安装

### 切换HOMEBREW源

```bash
export HOMEBREW_INSTALL_FROM_API=1
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
```

### 设置Golang国内代理
```bash
export GOPROXY=https://goproxy.cn,direct
```

### 安装Hugo
```bash
brew install hugo
```

## 常用命令
```bash
# 查看hugo版本
$ hugo version
hugo v0.115.1+extended darwin/amd64 BuildDate=unknown

# 创建一个新的网站
$ hugo new site TraceThePeaks

# 创建第一篇文章
$ hugo new posts/first_post.md

# 本地启动，默认是deployment环境运行，绑定的是127.0.0.1/localhost
$ hugo serve -D

# 本地调试，改为生产环境运行
$ hugo serve -e production --bind 0.0.0.0
```

## 创建文章
```bash
$ cd dihaifeng.github.io/

# 创建新文章
$ hugo new posts/second_post.md

# 写完文章可以本地环境预览
$ hugo serve -D

# 生成发布文件
$ hugo -DEF

# 提交git仓库
$ git add 
$ git commit -m 'add second_post.md'
$ git push origin main
```
> 通过new创建的新文章引用了默认的模块，里面包含了`draft: true`草稿标志，正式发布是要去除掉

##  自动发布
本博客源码仓库和博客仓库在一起维护
### 创建Git Page Action文件
```yaml
# 文件位置：dihaifeng.github.io/.github/workflows/hugo.yml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

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
      HUGO_VERSION: 0.115.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
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
        uses: actions/deploy-pages@v2
```

### 设置仓库为Git Pages
本博客`Build and deployment`使用的是`github actions Beta`自动生成的hugo.yaml进行自动发布。
