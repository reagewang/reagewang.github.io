---
layout: post
title: jetson nano板制作文件系统记录
tags: [jetson nano,filesystem]
comments: true
---

# 在ubuntu主机交叉编译L4T软件包
## 下载L4T源码
```
下载地址:https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/Sources/T210/public_sources.tbz2
```
## 解压和提取内核目录
```
tar -xjf public_sources.tbz2

cd public_sources/

tar -xjf kernel_src.tbz2

tree -L 3 public_sources/

public_sources/
├── hardware
│   └── nvidia
│       ├── platform
│       └── soc
├── kernel
│   ├── kernel-4.9
│   │   ├── arch
│   │   ├── block
│   │   ├── build.config.cuttlefish.x86_64
│   │   ├── build.config.goldfish.arm
│   │   ├── build.config.goldfish.arm64
│   │   ├── build.config.goldfish.mips
│   │   ├── build.config.goldfish.mips64
│   │   ├── build.config.goldfish.x86
│   │   ├── build.config.goldfish.x86_64
│   │   ├── certs
│   │   ├── COPYING
│   │   ├── CREDITS
│   │   ├── crypto
│   │   ├── Documentation
│   │   ├── drivers
│   │   ├── firmware
│   │   ├── fs
│   │   ├── include
│   │   ├── init
│   │   ├── ipc
│   │   ├── Kbuild
│   │   ├── Kconfig
│   │   ├── kernel
│   │   ├── lib
│   │   ├── MAINTAINERS
│   │   ├── Makefile
│   │   ├── mm
│   │   ├── modules.builtin
│   │   ├── modules.order
│   │   ├── Module.symvers
│   │   ├── net
│   │   ├── NVIDIA-REVIEWERS
│   │   ├── README
│   │   ├── REPORTING-BUGS
│   │   ├── rt-patches
│   │   ├── samples
│   │   ├── scripts
│   │   ├── security
│   │   ├── signing_key
│   │   ├── signing_key.x509
│   │   ├── sound
│   │   ├── System.map
│   │   ├── tools
│   │   ├── usr
│   │   ├── verity_dev_keys.x509
│   │   ├── virt
│   │   ├── vmlinux
│   │   └── vmlinux.o
│   ├── nvgpu
│   │   ├── drivers
│   │   ├── include
│   │   ├── Makefile.umbrella.tmk
│   │   ├── NVIDIA-REVIEWERS
│   │   ├── scripts
│   │   └── userspace
│   └── nvidia
│       ├── arch
│       ├── Documentation
│       ├── drivers
│       ├── include
│       ├── NVIDIA-REVIEWERS
│       ├── scripts
│       ├── security
│       └── sound
```
# 构建NVIDIA内核
## 安装预装软件
```
sudo apt install build-essential bc
```
## 安装L4T工具链
工具链包含以下组件：
* GCC版本：7.3.1
* Binutils版本：2.28.2.20170706
* Glibc版本：2.25
### 下载和解压工具链
```
wget http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz

sudo tar -xvf gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz -C /opt
```
### 修改环境变量
```
vim  ~/.bashrc

export PATH=/opt/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/:$PATH

source ~/.bashrc
```
### 导出相关环境变量
```
# 仅当前终端有效
export LOCALVERSION=-tegra
export CROSS_COMPILE=aarch64-linux-gnu-
```
### 创建.config 文件
```
cd kernel/kernel-4.9
make ARCH=arm64 tegra_defconfig
```
### 编译内核
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```
编译得到的内核为arch/arm64/boot/Image
### 安装Linux_for_Tegra
# 烧录内核
## 下载L4T Driver Package (BSP)
```
下载地址:https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/t210ref_release_aarch64/Tegra210_Linux_R32.4.3_aarch64.tbz2
```
### 解压文件
```
tar -xjf Jetson-210_Linux_R32.2.1_aarch64.tbz2

Linux_for_Tegra/
├── apply_binaries.sh
├── bootloader
├── build_l4t_bup.sh
├── create-jetson-nano-sd-card-image.sh
├── flash.sh
├── jetson-nano-emmc.conf -> p3448-0000-emmc.conf
├── jetson-nano-qspi.conf -> p3448-0000.conf
├── jetson-nano-qspi-sd.conf -> p3448-0000-sd.conf
├── jetson-tx1.conf -> p2371-2180-devkit.conf
├── kernel
├── l4t_generate_soc_bup.sh
├── lib
├── nv_tegra
├── p2371-2180-devkit-24x7.conf
├── p2371-2180-devkit.conf
├── p3448-0000.conf
├── p3448-0000.conf.common
├── p3448-0000-emmc.conf
├── p3448-0000-sd.conf
├── rootfs
├── source_sync.sh
└── TX1_boot-firmware-redundancy.txt
```
## 下载Sample Root Filesystem
```
下载地址:https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/t210ref_release_aarch64/Tegra_Linux_Sample-Root-Filesystem_R32.4.3_aarch64.tbz2
```
### 将镜像加压到Linux_for_Tegra/rootfs目录下
```
sudo tar -xjf Tegra_Linux_Sample-Root-Filesystem_R32.2.1_aarch64.tbz2 -C Linux_for_Tegra/rootfs

cd Linux_for_Tegra/
sudo ./apply_binaries.sh
```
### 注意：必须以sudo运行解压和apply_binaries.sh
## 将BSP软件烧录到jetson nano module板子
### 使jetson nano module进入recovery模式，接上usb到ubuntu 主机
```
lsusb
Bus 001 Device 057: ID 0955:7f21 NVidia Corp.
```
#### pid关联的板子
1. 7f21 for Jetson Nano (P3448, included in the developer kit)
2. 7f21 for Jetson Nano (P3448-0020, for production devices)
3. 7019 for Jetson AGX Xavier
4. 7c18 for Jetson TX2
5. 7018 for Jetson TX2i
6. 7418 for Jetson TX2 4GB
7. 7721 Jetson TX1

#### 根据board执行烧录命令
```
sudo ./flash.sh jetson-nano-emmc mmcblk0p1
```
#### 每一块板对应的配置文件
## 覆盖L4T Driver Package (BSP) Sources编译结果
```
cd kernel/kernel-4.9/

# 内核
cp kernel/kernel-4.9/arch/arm64/boot/Image Linux_for_Tegra/kernel/Image

# 设备树
cp arch/arm64/boot/Image ../../../Linux_for_Tegra/kernel/Image

# 内核模块
sudo make ARCH=arm64 modules_install INSTALL_MOD_PATH=../../../Linux_for_Tegra/rootfs/
```
# 在jetson nano上直接编译内核
将public_sources.tbz2复制到nano板上并解压。
## 编译内核
```
$ cd ~/kernel/kernel-4.9
$ zcat /proc/config.gz > .config

# Prepare the kernel build
$ make prepare
$ make modules_prepare

# Compile kernel image and kernel modules
$ time make -j5 Image
$ time make -j5 modules
```
## 安装内核与内核模块
```
# Install modules and kernel image
$ sudo make modules_install
$ sudo cp arch/arm64/boot/Image /boot/Image
```