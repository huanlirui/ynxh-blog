【五】Tauri 入门篇 - 跨平台编译

# 原创 lencx 浮之静 2022-07-25 00:37 发表于上海

背景
Tauri 严重依赖原生库和工具链，因此目前无法在某一平台实现交叉编译。最佳选择是使用托管在 GitHub Action[1]、Azure Pipelines[2]、GitLab[3] 或其他选项上的 CI/CD 管道进行编译。管道可以同时为每个平台运行编译，使编译和发布过程更加容易。

为了便于设置，官方目前提供 Tauri Action[4]。这是一个 GitHub Action，可在所有支持的平台上运行，编译软件，生成应用程序安装包，并将发布到 GitHub Releases[5]。

GitHub Action
从构思到生产，自动化工作流程

利用 GitHub Actions，在你的仓库中自动化、定制和执行你的软件开发工作流程。你可以发现、创建和共享操作，以执行你想要的任何工作，包括 CI/CD，并在一个完全定制的工作流程中组合操作。

使用 Action
创建 release.yml
在项目根路径下创建 .github/workflows 目录，在 .github/workflows 下创建 release.yml（文件名自定义） 文件。将以下内容复制到文件中：

# 可选，将显示在 GitHub 存储库的“操作”选项卡中的工作流名称

name: Release CI

# 指定此工作流的触发器

on:
push: # 匹配特定标签 (refs/tags)
tags: - 'v*' # 推送事件匹配 v*, 例如 v1.0，v20.15.10 等来触发工作流

# 需要运行的作业组合

jobs:

# 任务：创建 release 版本

create-release:
runs-on: ubuntu-latest
outputs:
RELEASE_UPLOAD_ID: ${{ steps.create_release.outputs.id }}

    steps:
      - uses: actions/checkout@v2
      # 查询版本号（tag）
      - name: Query version number
        id: get_version
        shell: bash
        run: |
          echo "using version tag ${GITHUB_REF:10}"
          echo ::set-output name=version::"${GITHUB_REF:10}"

      # 根据查询到的版本号创建 release
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: '${{ steps.get_version.outputs.VERSION }}'
          release_name: 'app ${{ steps.get_version.outputs.VERSION }}'
          body: 'See the assets to download this version and install.'

# 编译 Tauri

build-tauri:
needs: create-release
strategy:
fail-fast: false
matrix:
platform: [macos-latest, ubuntu-latest, windows-latest]

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v2

     # 安装 Node.js
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: 16

      # 安装 Rust
      - name: Install Rust stable
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable

      # 使用 Rust 缓存，加快安装速度
      - uses: Swatinem/rust-cache@v1

      - name: install dependencies (ubuntu only)
        if: matrix.platform == 'ubuntu-latest'
        run: |
          sudo apt-get update
          sudo apt-get install -y libgtk-3-dev webkit2gtk-4.0 libappindicator3-dev librsvg2-dev patchelf

      # 可选，如果需要将 Rust 编译为 wasm，则安装 wasm-pack
      - uses: jetli/wasm-pack-action@v0.3.0
        with:
          # Optional version of wasm-pack to install(eg. 'v0.9.1', 'latest')
          version: v0.9.1

      # 可选，如果需要使用 rsw 构建 wasm，则安装 rsw
      - name: Install rsw
        run: cargo install rsw

      # 获取 yarn 缓存路径
      - name: Get yarn cache directory path
        id: yarn-cache-dir-path
        run: echo "::set-output name=dir::$(yarn config get cacheFolder)"

      # 使用 yarn 缓存
      - name: Yarn cache
        uses: actions/cache@v2
        id: yarn-cache # use this to check for `cache-hit` (`steps.yarn-cache.outputs.cache-hit != 'true'`)
        with:
          path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      # 安装依赖执行构建，以及推送 github release
      - name: Install app dependencies and build it
        run: yarn && yarn build
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          releaseId: ${{ needs.create-release.outputs.RELEASE_UPLOAD_ID }}

具体可以参考 OhMyBox/.github/workflows/release.yml[6]

触发 Action
使用 GitHub Tag 来触发 Action

# 创建 tag

git tag v0.1.0

# 推送 tag

git push --tag
图片

图片

常见问题
Error: Resource not accessible by integration
release.yml 中的 GitHub 环境令牌 GITHUB_TOKEN 由 GitHub 为每个运行的工作流自动颁发，无需进一步配置，这意味着没有秘密泄露的风险。但是，此令牌在默认情况下仅具有读取权限，在运行工作流时可能会收到 Resource not accessible by integration（资源无法通过集成访问）错误。如果发生这种情况，需要为此令牌添加写入权限。为此，请前往 GitHub 仓库 Settings，然后选择 Actions，向下滚动到 Workflow permissions（工作流权限）并选中 Read and write permissions（读取和写入权限）。

Settings -> Actions -> General -> Workflow permissions -> Read and write permissions
Action 意外终止
如果 Action 因某些原因（如缓存错误，依赖下载失败等）终止工作流，不需要重新创建 tag 来触发工作流，可以通过 Re-run all jobs 按钮来重新触发。
