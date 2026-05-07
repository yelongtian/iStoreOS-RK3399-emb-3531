# iStoreOS-RK3399 Code Wiki

## 目录

1. [项目概述
2. [项目架构
3. [主要模块职责
4. [关键文件说明
5. [设备适配流程
6. [依赖关系
7. [项目运行方式
8. [开发指南

---

## 1. 项目概述

### 1.1 项目简介

**iStoreOS-RK3399** 是一个基于 iStoreOS 固件构建仓库，专门为 Rockchip RK3399 芯片的各类设备提供支持。iStoreOS 是一个入门级的路由系统和 NAS 系统，基于 OpenWRT 构建。

**主要特点：
- 支持多种 RK3399 设备的 iStoreOS 固件构建
- 提供 GitHub Actions 自动化构建流程
- 每个设备有独立的配置和补丁
- 非官方构建，但提供了丰富的设备支持

### 1.2 支持设备列表

| 序号 | 设备型号 | 说明 |
| ---- | ------ | ---- |
| 1 | am40 | SMART AM40 |
| 2 | dg3399 | DG3399 |
| 3 | dlfr100 | DLFR100 |
| 4 | fine3399 | FINE3399 |
| 5 | fmx1-pro | FMX1-PRO |
| 6 | fnet3399 | FNET3399 |
| 7 | h3399pc | H3399PC |
| 8 | king3399 | KING3399 |
| 9 | mpc1903 | MPC1903 |
| 10 | sv-33a6x | SV-33A6X |
| 11 | sv901-eaio | SV901-EAIO |
| 12 | tn3399 | TN3399 |
| 13 | tpm-312 | TPM-312 |
| 14 | tvi3315a | TVI3315A |
| 15 | xiaobao-nas | 小宝 NAS |
| 16 | zysj-1739a | ZYSJ-1739A |
| 17 | emb-3531 | EMB-3531 |
| 18 | emb3531 | EMB3531 (别名) |

### 1.3 默认登录信息

- **用户名**: `root`
- **密码**: `password`
- **网络配置**: 
  - 单网口设备: 该网口即为 `LAN`
  - 双网口设备: 一个 `WAN`，一个 `LAN`

---

## 2. 项目架构

### 2.1 整体架构图

```
iStoreOS-RK3399/
├── .github/
│   └── workflows/
│       └── build-istoreos.yml  # GitHub Actions 构建工作流
├── {device-name}/       # 各设备配置目录（如 am40/, dg3399/, ...)
│   ├── .config                          # OpenWRT 配置文件
│   ├── diy-part1.sh                     # 第一阶段 DIY 脚本
│   ├── diy-part2.sh                     # 第二阶段 DIY 脚本
│   ├── kernel-rockchip/
│   │   ├── patches/                     # 内核补丁
│   │   └── 02_network                  # 网络配置（部分设备）
│   └── uboot-rockchip/
│       ├── Makefile                  # U-Boot Makefile
│       ├── patches/                     # U-Boot 补丁
│       └── dts/                      # 设备树文件（部分设备）
├── depends/
│   └── ubuntu-22.04                   # Ubuntu 22.04 依赖包列表
├── feeds.conf.default             # OpenWRT Feeds 配置
├── README.md                      # 项目说明文档
└── istoreos.png                   # iStoreOS Logo
```

### 2.2 目录结构说明

| 目录/文件 | 功能说明 |
|--------|------|
| `.github/workflows/` | GitHub Actions 工作流配置目录 |
| `{device-name}/` | 设备配置目录，每个设备一个独立目录 |
| `depends/` | 依赖包列表目录 |
| `feeds.conf.default` | OpenWRT 软件源配置 |

---

## 3. 主要模块职责

### 3.1 GitHub Actions 构建系统

**文件**: [build-istoreos.yml](file:///workspace/.github/workflows/build-istoreos.yml)

**职责**:
- 自动化构建 iStoreOS 固件
- 支持多设备构建选择
- 集成编译环境初始化
- 固件编译和发布

**主要步骤**:

1. **准备阶段**
   - 检查服务器配置
   - 初始化编译环境
   - 克隆 iStoreOS 源码

2. **配置阶段**
   - 加载自定义 feeds
   - 更新和安装 feeds
   - 加载设备特定配置

3. **编译阶段**
   - 下载软件包
   - 编译固件
   - 清理并复制固件

4. **发布阶段**
   - 设置 Release 标签
   - 上传固件到 GitHub Releases
   - 删除旧版本固件

### 3.2 设备配置模块

每个设备配置目录包含以下关键组件：

#### 3.2.1 diy-part1.sh

**职责**:
- 修改固件版本信息
- 自定义作者信息
- 第一阶段的准备工作

**示例** ([am40/diy-part1.sh](file:///workspace/am40/diy-part1.sh)
```bash
# 修改版本为编译日期
date_version=$(date +"%Y%m")
echo $date_version > version

# 为iStoreOS固件版本加上编译作者
author="Lemon1151"
sed -i "s/DISTRIB_DESCRIPTION.*/DISTRIB_DESCRIPTION='%D %V ${date_version} by ${author}'/g" package/base-files/files/etc/openwrt_release
sed -i "s/OPENWRT_RELEASE.*/OPENWRT_RELEASE=\"%D %V ${date_version} by ${author}\"/g" package/base-files/files/usr/lib/os-release
```

#### 3.2.2 diy-part2.sh

**职责**:
- 移植设备支持
- 配置 U-Boot 和内核
- 复制设备特定的设备树和补丁

**示例** ([am40/diy-part2.sh](file:///workspace/am40/diy-part2.sh)
```bash
# 移植设备到 target/linux/rockchip/image/armv8.mk
echo -e "\\ndefine Device/smart_am40
  DEVICE_VENDOR := SMART
  DEVICE_MODEL := AM40
  SOC := rk3399
  UBOOT_DEVICE_NAME := am40-rk3399
  IMAGE/sysupgrade.img.gz := boot-common | boot-script | pine64-img | gzip | append-metadata
  DEVICE_PACKAGES := wpad-mbedtls kmod-rtw88-8822be ...
endef
TARGET_DEVICES += smart_am40" >> target/linux/rockchip/image/armv8.mk

# 复制修改好的uboot/Makefile到对应目录
cp -f $GITHUB_WORKSPACE/am40/uboot-rockchip/Makefile package/boot/uboot-rockchip/Makefile

# 复制patch到对应的目录
cp -f $GITHUB_WORKSPACE/am40/uboot-rockchip/patches/990-rk3399-am40-uboot.patch package/boot/uboot-rockchip/patches/990-rk3399-am40-uboot.patch
cp -f $GITHUB_WORKSPACE/am40/kernel-rockchip/patches/990-rockchip-rk3399-am40-kernel.patch target/linux/rockchip/patches-6.6/990-rockchip-rk3399-am40-kernel.patch
```

#### 3.2.3 .config

**职责**:
- OpenWRT 构建配置
- 包含目标设备配置
- 软件包选择

#### 3.2.4 kernel-rockchip/patches/

**职责**:
- 内核补丁目录
- 包含设备特定的内核补丁

#### 3.2.5 uboot-rockchip/

**职责**:
- U-Boot 配置
- U-Boot 补丁
- 设备树文件 (部分设备)

### 3.3 Feeds 配置模块

**文件**: [feeds.conf.default](file:///workspace/feeds.conf.default)

**职责**: 定义 OpenWRT 软件源

**内容**:
```
src-git packages https://github.com/jjm2473/packages.git;istoreos-24.10
src-git luci https://github.com/jjm2473/luci.git;istoreos-24.10
src-git routing https://github.com/openwrt/routing.git;openwrt-24.10
src-git telephony https://github.com/openwrt/telephony.git;openwrt-24.10
src-git store https://github.com/linkease/istore.git;main
src-git third https://github.com/jjm2473/openwrt-third.git;main
src-git linkease_nas https://github.com/linkease/nas-packages.git;master
src-git linkease_nas_luci https://github.com/linkease/nas-packages-luci.git;main
src-git oaf https://github.com/jjm2473/OpenAppFilter.git;dev4
```

### 3.4 依赖管理模块

**文件**: [depends/ubuntu-22.04](file:///workspace/depends/ubuntu-22.04)

**职责**: 列出 Ubuntu 22.04 编译环境所需依赖包

---

## 4. 关键文件说明

### 4.1 build-istoreos.yml

**位置**: [.github/workflows/build-istoreos.yml](file:///workspace/.github/workflows/build-istoreos.yml)

**关键功能**:
- 工作流触发条件: `repository_dispatch` 和 `workflow_dispatch`
- 支持设备选择输入
- 多分支构建 (istoreos-24.10)
- 架构: aarch64_cortex-a53
- 编译环境: Ubuntu 22.04

**主要环境变量**:
```yaml
REPO_URL: https://github.com/istoreos/istoreos
FEEDS_CONF: feeds.conf.default
CONFIG_FILE: ${{ inputs.board }}/.config
DIY_P1_SH: ${{ inputs.board }}/diy-part1.sh
DIY_P2_SH: ${{ inputs.board }}/diy-part2.sh
```

### 4.2 diy-part1.sh 通用结构

所有设备的 `diy-part1.sh` 基本结构相同，主要负责：
1. 设置版本号
2. 修改作者信息

### 4.3 diy-part2.sh 通用结构

所有设备的 `diy-part2.sh` 主要负责：
1. 在 `target/linux/rockchip/image/armv8.mk` 中添加设备定义
2. 复制 U-Boot Makefile
3. 复制 U-Boot 补丁
4. 复制内核补丁
5. 部分设备可能还会复制网络配置

### 4.4 feeds.conf.default

定义了以下软件源：
- packages: iStoreOS 软件包
- luci: iStoreOS LuCI 界面
- routing: 路由协议
- telephony: 电话相关
- store: iStore 应用商店
- third: 第三方软件包
- linkease_nas: NAS 相关软件包
- linkease_nas_luci: NAS 相关 LuCI 界面
- oaf: OpenAppFilter 应用过滤

---

## 5. 设备适配流程

### 5.1 添加新设备的步骤

1. **创建设备目录**:
```
mkdir -p new-device/kernel-rockchip/patches
mkdir -p new-device/uboot-rockchip/patches
```

2. **创建配置文件**:
   - `new-device/.config`
   - `new-device/diy-part1.sh`
   - `new-device/diy-part2.sh`
   - `new-device/kernel-rockchip/patches/xxx.patch`
   - `new-device/uboot-rockchip/Makefile`
   - `new-device/uboot-rockchip/patches/xxx.patch`

3. **修改 diy-part2.sh**:
   - 添加设备定义到 `target/linux/rockchip/image/armv8.mk`
   - 配置设备特定的软件包
   - 复制补丁文件

4. **在 GitHub Actions 工作流中添加设备选项**:
   - 修改 [build-istoreos.yml](file:///workspace/.github/workflows/build-istoreos.yml) 中的 `inputs.board.options`

### 5.2 设备定义结构

```makefile
define Device/{vendor}_{model}
  DEVICE_VENDOR := {Vendor Name}
  DEVICE_MODEL := {Model Name}
  SOC := rk3399
  UBOOT_DEVICE_NAME := {device-name}-rk3399
  IMAGE/sysupgrade.img.gz := boot-common | boot-script | pine64-img | gzip | append-metadata
  DEVICE_PACKAGES := {package1} {package2} ...
endef
TARGET_DEVICES += {vendor}_{model}
```

---

## 6. 依赖关系

### 6.1 外部依赖

**iStoreOS 源码**:
- 仓库: https://github.com/istoreos/istoreos
- 分支: istoreos-24.10

**Feeds 仓库**:
- packages: https://github.com/jjm2473/packages.git
- luci: https://github.com/jjm2473/luci.git
- routing: https://github.com/openwrt/routing.git
- telephony: https://github.com/openwrt/telephony.git
- store: https://github.com/linkease/istore.git
- third: https://github.com/jjm2473/openwrt-third.git
- linkease_nas: https://github.com/linkease/nas-packages.git
- linkease_nas_luci: https://github.com/linkease/nas-packages-luci.git
- oaf: https://github.com/jjm2473/OpenAppFilter.git

### 6.2 编译环境依赖

Ubuntu 22.04 依赖包列表:
- ack, antlr3, asciidoc
- autoconf, automake, autopoint
- binutils, bison, build-essential
- bzip2, ccache, cmake
- cpio, curl, device-tree-compiler
- fastjar, flex, gawk
- gettext, gcc-multilib, g++-multilib
- git, gperf, haveged
- help2man, intltool
- libc6-dev-i386, libelf-dev
- libglib2.0-dev, libgmp3-dev
- libltdl-dev, libmpc-dev
- libmpfr-dev, libncurses5-dev
- libncursesw5-dev, libreadline-dev
- libssl-dev, libtool
- lrzsz, mkisofs, msmtp
- nano, ninja-build, p7zip
- p7zip-full, patch, pkgconf
- python2, python3.6, python3
- python3-pyelftools, libpython3-dev
- qemu-utils, rsync, scons
- squashfs-tools, subversion
- swig, texinfo, uglifyjs
- upx-ucl, unzip, vim
- wget, xmlto, xxd
- zlib1g-dev

### 6.3 Python 依赖

- pyelftools

---

## 7. 项目运行方式

### 7.1 GitHub Actions 构建

1. **触发构建**:
   - 访问 GitHub 仓库
   - 进入 Actions 标签页
   - 选择 "Build iStore OS" 工作流
   - 点击 "Run workflow"
   - 选择要构建的设备
   - 可选：启用 SSH 连接调试

2. **构建产物**:
   - 固件文件: *.img.gz
   - SHA256 校验文件: *.sha
   - 自动发布到 GitHub Releases

### 7.2 本地构建

虽然项目主要设计为使用 GitHub Actions 构建，但也可以本地构建：

```bash
# 1. 安装依赖
sudo apt-get update
sudo apt-get install $(cat depends/ubuntu-22.04)

# 2. 克隆源码
git clone https://github.com/istoreos/istoreos -b istoreos-24.10 openwrt
cd openwrt

# 3. 配置 feeds
cp ../feeds.conf.default feeds.conf.default

# 4. 运行 diy-part1.sh (选择设备，例如 am40)
cp ../am40/diy-part1.sh .
chmod +x diy-part1.sh
./diy-part1.sh

# 5. 更新和安装 feeds
./scripts/feeds update -a
./scripts/feeds install -a

# 6. 加载配置
cp ../am40/.config .config
cp ../am40/diy-part2.sh .
chmod +x diy-part2.sh
./diy-part2.sh

# 7. 编译
make defconfig
make download -j8
make -j$(nproc)
```

---

## 8. 开发指南

### 8.1 代码规范

- Shell 脚本:
  - 使用 `#!/bin/bash` 作为脚本开头
  - 遵循 MIT 许可证声明
  - 添加必要的注释

- 补丁文件:
  - 命名规范: `{序号}-{芯片}-{设备}-{组件}.patch
  - 清晰的补丁描述

### 8.2 提交规范

- 清晰的提交信息
- 描述修改内容
- 关联相关设备

### 8.3 常见问题

1. **WiFi 不可用**:
   - 访问 https://github.com/armbian/firmware
   - 找到对应的无线网卡驱动
   - 复制到对应目录替换

2. **无法启动**:
   - 连接 TTL 串口
   - 查看启动日志

3. **编译失败**:
   - 检查 .config 配置
   - 查看编译日志
   - 尝试使用 `make -j1 V=s` 查看详细输出

---

## 鸣谢

- [istoreos](https://github.com/istoreos/istoreos)
- [P3TERX/Actions-OpenWrt](https://github.com/P3TERX/Actions-OpenWrt)
- [xiaomeng9597](https://github.com/xiaomeng9597)
- [cm9vdA](https://github.com/cm9vdA/build-linux)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [Lean's OpenWrt](https://github.com/coolsnowwolf/lede)

---

## 免责声明

- 本固件仅供学习研究，严禁用于任何商业用途
- 使用本固件产生的所有后果均由使用者自行承担
- 固件可能存在bug，开发者不提供任何形式的技术支持
- 请严格遵守国家网络安全法律法规，合法使用

