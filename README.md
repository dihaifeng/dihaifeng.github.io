# dihaifeng.github.io

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
hugo version

# 创建一个新的网站
hugo new site TraceThePeaks

# 创建第一篇文章
hugo new posts/first_post.md

# 本地调试，默认是deployment环境运行，绑定的是127.0.0.1/localhost
hugo serve

# 本地调试，改为生产环境运行 
hugo serve -e production --bind 0.0.0.0
```

### 发布博客
本博客`Build and deployment`使用的是`github actions Beta`自动生成的hugo.yaml进行自动发布。
