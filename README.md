# Fastfetch for OpenWrt

为 OpenWrt 构建的 [fastfetch](https://github.com/fastfetch-cli/fastfetch) 软件包 —— 快速、轻量的系统信息工具，可替代 `neofetch`。

[![Release](https://img.shields.io/github/v/release/mineextremely/Fastfetch4Openwrt?label=release)](https://github.com/mineextremely/Fastfetch4Openwrt/releases)
[![Build](https://img.shields.io/github/actions/workflow/status/mineextremely/Fastfetch4Openwrt/build.yml?label=build)](https://github.com/mineextremely/Fastfetch4Openwrt/actions/workflows/build.yml)
[![License](https://img.shields.io/github/license/mineextremely/Fastfetch4Openwrt)](LICENSE)

## 特性

- 只打包 `fastfetch` 本体（不含 `flashfetch`），安装后是单个 `/usr/bin/fastfetch`
- 面向路由器裁剪：不编译 X11、Wayland、DRM、PulseAudio、OpenGL、Vulkan、systemd、SQLite3、ImageMagick 等依赖
- 同时支持 OpenWrt 24.10（opkg / `.ipk`）和 25.12（apk / `.apk`）
- GitHub Actions 预编译 3 种架构 × 2 个 SDK 版本
- 提供带签名的 opkg/apk 软件源，可直接 `opkg install fastfetch` / `apk add fastfetch`
- 版本号集中管理，每日自动检查上游新版本

## 仓库结构

```text
.
├── fastfetch/
│   ├── Makefile          # OpenWrt 包定义：源码地址、SHA256、编译选项
│   └── version.mk        # fastfetch 版本号（唯一来源）
├── keys/
│   ├── key-build.pub     # opkg（usign）公钥
│   └── public-key.pem    # apk（EC P-256）公钥
├── .github/workflows/
│   ├── auto-update.yml   # 每日检查上游 → 更新版本 → PR → 自动合并 → 触发构建
│   └── build.yml         # 多架构编译 + 发布 Release + 发布签名软件源
├── README.md
└── LICENSE
```

签名软件源发布在独立的 `feed` 分支，由 GitHub Pages 提供：
<https://mineextremely.github.io/Fastfetch4Openwrt/>

## 版本管理

`fastfetch/version.mk` 是版本号的唯一来源：

```make
FASTFETCH_VERSION=2.70.0
```

`fastfetch/Makefile` 通过 `-include` 读取它，并保留兜底值：

```make
PKG_VERSION:=$(or $(FASTFETCH_VERSION),2.70.0)
PKG_HASH:=8aa01bd3faff2be1d94f231d76489ce6d9f781f51f6e93eafe82359d6d8669af
```

- 正常构建时版本号以 `version.mk` 为准，`$(or ...)` 里的值只在没有 `version.mk` 时生效
- `PKG_HASH` 对应 `https://codeload.github.com/fastfetch-cli/fastfetch/tar.gz/<版本>` 的 SHA256

手动升级时同步修改 `version.mk` 的版本号、`Makefile` 的兜底版本与 `PKG_HASH` 即可；每日自动更新流程做的也是这三件事。

## 编译

需要与目标固件匹配的 OpenWrt SDK（24.10.x 或 25.12.x）。跨大版本（24.10 ↔ 25.12）的包不能混用。

### 方式一：作为 feed 引入

```bash
cd /path/to/openwrt
echo "src-git fastfetch4openwrt https://github.com/mineextremely/Fastfetch4Openwrt.git" >> feeds.conf.default
./scripts/feeds update fastfetch4openwrt
./scripts/feeds install fastfetch
make menuconfig        # Utilities → fastfetch
make package/fastfetch/compile V=s
```

> 如果目录里已经有 `feeds.conf`，请改那个文件，而不是 `feeds.conf.default`。

### 方式二：直接拷进 package 目录

```bash
cp -r Fastfetch4Openwrt/fastfetch /path/to/openwrt-sdk/package/
cd /path/to/openwrt-sdk
make package/fastfetch/compile V=s
```

编译产物位于 `bin/packages/<架构>/<feed>/`，24.10 为 `fastfetch_*.ipk`，25.12 为 `fastfetch-*.apk`。

## 预编译包

在 [Releases](https://github.com/mineextremely/Fastfetch4Openwrt/releases) 下载。每个版本按
`fastfetch-<版本>-<架构>-<SDK>.zip` 命名，例如：

```text
fastfetch-2.70.0-x86_64-24.10.6.zip
```

zip 内是对应架构与 SDK 的安装包（`.ipk` 或 `.apk`）。

| 架构 | SDK 版本 | 包格式 |
|------|----------|--------|
| `aarch64_cortex-a53` | 24.10.6 / 25.12.3 | ipk / apk |
| `aarch64_generic` | 24.10.6 / 25.12.3 | ipk / apk |
| `x86_64` | 24.10.6 / 25.12.3 | ipk / apk |

### 安装

```bash
# OpenWrt 24.10（opkg）
opkg install fastfetch_*.ipk

# OpenWrt 25.12（apk）
apk add --allow-untrusted fastfetch-*.apk
```

```bash
fastfetch
```

> 直接安装 Release 里的单个 `.apk` 时，因为包文件本身没有签名，需要 `--allow-untrusted`；想免去这个参数并让包管理器自动校验，请使用下面的签名软件源。

## 签名软件源（推荐）

构建时用固定密钥签名 `Packages` / `packages.adb` 索引，公钥随仓库发布。把对应版本的源加入路由器后，可以直接按包名安装并享受签名校验。

OpenWrt 24.10（opkg）：

```bash
wget -O /tmp/key-build.pub https://mineextremely.github.io/Fastfetch4Openwrt/key-build.pub
opkg-key add /tmp/key-build.pub
. /etc/openwrt_release
echo "src/gz fastfetch4openwrt https://mineextremely.github.io/Fastfetch4Openwrt/24.10/${DISTRIB_ARCH}" >> /etc/opkg/customfeeds.conf
opkg update
opkg install fastfetch
```

OpenWrt 25.12（apk）：

```bash
wget -O /etc/apk/keys/fastfetch.pem https://mineextremely.github.io/Fastfetch4Openwrt/public-key.pem
. /etc/openwrt_release
echo "https://mineextremely.github.io/Fastfetch4Openwrt/25.12/${DISTRIB_ARCH}/packages.adb" >> /etc/apk/repositories.d/customfeeds.list
apk update
apk add fastfetch
```

- 源目录按 SDK 大版本分开：24.10 用 `Packages` / `.ipk`，25.12 用 `packages.adb` / `.apk`，不能混用
- `${DISTRIB_ARCH}` 来自 `/etc/openwrt_release`，例如 `aarch64_cortex-a53`、`x86_64`
- 公钥也可直接从仓库的 `keys/` 目录获取；usign 指纹由 `opkg-key add` 自动计算

## 自动化流程

### 每日自动更新

`auto-update.yml` 每天 00:00 UTC（北京时间 08:00）运行一次：

```text
检查 fastfetch 上游最新 Release
 ├─ 与 version.mk 中的版本相同 → 直接结束
 └─ 有新版本
      ├─ 下载 tarball 并计算 SHA256
      ├─ 更新 version.mk 和 Makefile
      ├─ 推送 auto-update-fastfetch-<版本> 分支
      ├─ 创建 PR（已存在则复用）
      ├─ 自动合并到默认分支
      └─ 触发 build.yml 编译并发布
```

- 只有版本号变化才会开 PR，不会每天重复提交
- 合并带 6 次重试（GitHub 计算 mergeable 有延迟）
- PR 由 `GITHUB_TOKEN` 创建，不会触发 `pull_request` 类工作流，所以构建是显式 `workflow_dispatch` 的
- 也可手动触发：Actions → **Auto-Update Fastfetch** → Run workflow

### 手动编译发布

Actions → **Build-OpenWrt-fastfetch** → Run workflow，可填写：

| 输入 | 说明 |
|------|------|
| `tag` | Release 版本号，留空则使用 `version.mk` 中的版本 |
| `text` | 追加到 Release 说明末尾的文字 |

矩阵为 3 架构 × 2 SDK，全部完成后把产物打包成 zip 并创建/更新对应 tag 的 Release，同时把各架构的签名索引与安装包发布到 `feed` 分支。

## 维护说明

- 默认分支为 `auto-update-fastfetch`
- 自动更新与发布使用仓库自带的 `GITHUB_TOKEN`，所需权限：`contents: write`、`pull-requests: write`、`actions: write`
- 签名私钥只存放在 Actions Secrets：`KEY_BUILD`（usign）与 `PRIVATE_KEY`（apk EC P-256）；公钥提交在 `keys/` 并复制到 `feed` 分支根目录
- 软件源由 `build.yml` 的 `publish-feed` 任务发布到 `feed` 分支，经 GitHub Pages 提供；24.10/25.12 的目录必须严格分开
- 若默认分支启用 required status checks，`gh pr merge` 会失败，需要调整分支保护或改用 PAT
- `GitHubActionsLogs/` 是本地排查 Actions 日志的目录，已加入 `.gitignore`，不会提交

## 故障排查

| 现象 | 处理 |
|------|------|
| 编译报 `Hash mismatch` | 确认 `version.mk` 的版本与 `PKG_HASH` 对应；重新对官方 tarball 计算 SHA256 |
| `apk` 安装提示签名错误 | 本地单包安装可用 `apk add --allow-untrusted`；从本仓库签名源安装不应出现，若出现请确认公钥路径与 `packages.adb` 地址 |
| 安装后无法运行或依赖缺失 | 包架构必须与固件一致，可用 `opkg print-architecture` 确认 |
| 自动更新 PR 未合并、Actions 变红 | 查看运行日志，确认默认分支没有阻止 bot 的合并规则 |

## 许可证

[MIT](LICENSE)
