# zola-deploy-chinese-indexing

由于 `zola` 默认不支持中文索引，因此使用 GitHub Action 部署时需要重新手动编译，这里提供一套解决方案

- 首先确保 `GitHub` 的仓库开启了 `Actions` 的写入权限，具体为找到仓库的 `Settings - Actions - General` ，滚动到 `Workflow permissions` 中，选择 `Read and write permissions` 。
- 可以参考以下 Action 代码，放置到 `.github/workflow` 中

``` yml
# .github/workflows/your-action-file.yml

name: Zola on GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build:
    name: Publish site
    runs-on: ubuntu-latest
    steps:
      # 1. 检出你的博客源码
      - name: Checkout main
        uses: actions/checkout@v4
        with:
          submodules: true

      # 2. 安装 Rust 环境
      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      # 3. 缓存 Cargo 全局目录和 Zola 可执行文件
      # 注意：我们移除了 `target/`，因为它对 `cargo install` 无效
      # 缓存的 key 可以是静态的，因为我们总是安装同一个 Zola 版本
      - name: Cache Cargo and Zola binary
        id: cache-zola # 给这个步骤一个 id，方便后面引用
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/bin/zola
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
          key: ${{ runner.os }}-cargo-zola-with-chinese-indexing

      # 4. 编译并安装 Zola (仅在缓存未命中时运行)
      # - 使用 `id: cache-zola` 的输出来判断是否需要安装
      # - 移除了 `--force`，否则每次都会强制重新安装，让缓存失效
      - name: Install Zola with Chinese indexing
        if: steps.cache-zola.outputs.cache-hit != 'true'
        run: cargo install zola --git https://github.com/getzola/zola.git --features indexing-zh

      # 5. 构建 Zola 网站
      - name: Build site
        run: zola build

      # 6. 部署到 GitHub Pages
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

- 将 `config.toml` 中的 `default_language` 改为 `zh` （不能用 `zh-Hans` ！）


