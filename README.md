# Fastfetch for OpenWrt

为 OpenWrt 构建的 [fastfetch](https://github.com/fastfetch-cli/fastfetch) 软件包 —— 一个快速、轻量的 neofetch 替代工具。

## 使用

将本仓库添加到 OpenWrt SDK 或 feeds 中，然后编译：

```bash
# 放入 SDK 的 package 目录
cp -r Fastfetch4Openwrt /path/to/openwrt-sdk/package/fastfetch

# 编译
cd /path/to/openwrt-sdk
make package/fastfetch/compile
```

## 预编译包

在 [Releases](https://github.com/mineextremely/Fastfetch4Openwrt/releases) 页面下载对应架构的预编译包：

- **24.10.x** —— ipk 安装包
- **25.12.x** —— apk 安装包

### 安装

```bash
# ipk
opkg install fastfetch_*.ipk

# apk（跳过证书验证）
apk add --allow-untrusted fastfetch-*.apk
```

## 支持架构

| 架构 | SDK |
|------|-----|
| aarch64_cortex-a53 | 24.10.6 / 25.12.3 |
| aarch64_generic | 24.10.6 / 25.12.3 |
| x86_64 | 24.10.6 / 25.12.3 |

## 版本

当前 fastfetch 版本：`2.63.0`（见 [version.mk](version.mk)）

## 许可证

MIT
