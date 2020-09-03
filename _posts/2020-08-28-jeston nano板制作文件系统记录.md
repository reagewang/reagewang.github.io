---
layout: post
title: jetson nano板制作文件系统记录
tags: [jetson nano,filesystem]
comments: true
---


在ubuntu主机交叉编译L4T软件包

## [下载L4T Driver Package (BSP) Sources源码](https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/Sources/T210/public_sources.tbz2)

## 解压和提取内核目录

```bash
tar -xjf public_sources.tbz2

cd Linux_for_Tegra/sources/public/

tar -xjf kernel_src.tbz2

tree -L 3 Linux_for_Tegra/sources/public/

Linux_for_Tegra/sources/public/
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
> sudo apt install build-essential bc

## 安装L4T工具链
工具链包含以下组件：
* GCC版本：7.3.1
* Binutils版本：2.28.2.20170706
* Glibc版本：2.25

### [下载和解压工具链](http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz)
```shell
wget http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz

sudo tar -xvf gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz -C /opt
```

### 修改环境变量
> vim  ~/.bashrc
> 
> export PATH=/opt/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/:$PATH
> 
> source ~/.bashrc

### 设置相关环境变量
```
# 仅当前终端有效
export PATH=/opt/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin:/home/wh/bin:/usr/sbin:/usr/bin:/bin:/sbin:/home/wh/.local/bin
export LOCALVERSION=-tegra
export CROSS_COMPILE=aarch64-linux-gnu-
```
### 创建.config 文件

> cd kernel/kernel-4.9
> 
> make ARCH=arm64 tegra_defconfig
```
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  SHIPPED scripts/kconfig/zconf.tab.c
  SHIPPED scripts/kconfig/zconf.lex.c
  SHIPPED scripts/kconfig/zconf.hash.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
```
### 编译内核
> make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```
编译得到的内核为arch/arm64/boot/Image
```
### 拷贝到Linux_for_Tegra
```
cp Linux_for_Tegra/source/public/kernel/kernel-4.9/arch/arm64/boot/Image Linux_for_Tegra/kernel/Image
```
# 烧录内核
## [下载L4T Driver Package (BSP)](https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/t210ref_release_aarch64/Tegra210_Linux_R32.4.3_aarch64.tbz2)
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
## [下载Sample Root Filesystem](https://developer.nvidia.com/embedded/L4T/r32_Release_v4.3/t210ref_release_aarch64/Tegra_Linux_Sample-Root-Filesystem_R32.4.3_aarch64.tbz2)
### 将镜像解压到Linux_for_Tegra/rootfs目录下
```
sudo tar -xjf Tegra_Linux_Sample-Root-Filesystem_R32.2.1_aarch64.tbz2 -C Linux_for_Tegra/rootfs

cd Linux_for_Tegra/
sudo ./apply_binaries.sh
```
> `注意：必须以sudo运行解压和apply_binaries.sh`
## 将BSP软件烧录到jetson nano module板子
> 使jetson nano module进入recovery模式，接上usb到ubuntu 主机
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
> sudo ./flash.sh jetson-nano-emmc mmcblk0p1
#### 每一块板对应的配置文件
## 覆盖L4T Driver Package (BSP) Sources编译结果
```
cd kernel/kernel-4.9/

# 内核
cp arch/arm64/boot/Image /home/wh/nfs/Linux_for_Tegra/kernel/Image

# 设备树
cp arch/arm64/boot/dts /home/wh/nfs/Linux_for_Tegra/kernel/dts

# 内核模块
sudo make ARCH=arm64 modules_install INSTALL_MOD_PATH=/home/wh/nfs/Linux_for_Tegra/rootfs/
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
# Tegra_Linux_Sample-Root-Filesystem_R32.4.3_aarch64.tbz2解压缩文件夹大小
```
/bin 11.3M
/dev 
/etc 6.9M
/home 

/lib 357.7M
/lib/firmware 321M
/lib/aarch64-linux-gnu 18M

/media 
/mnt
/opt
/proc
/root
/run 115.5KB
/sbin 11.2M
/snap
/srv
/sys
/tmp

/usr 3.3GB
/usr/bin 401.6M
/usr/sbin 22.7M
/usr/include 29.8M
/usr/lib 2.1GB
/usr/share 802.8M

/usr/lib/libreoffice 273M
/usr/lib/debug 35M
/usr/lib/python3.6 22M
/usr/lib/chromium-browser 232M
/usr/lib/gcc 67M
/usr/lib/git-core 20M
/usr/lib/thunderbird 154M
/usr/lib/aarch64-linux-gnu 900M
/usr/lib/python2.7 29M
/usr/lib/python3 129M
/usr/lib/snapd 45M

/usr/share/libreoffice 14M
/usr/share/example-content 13M
/usr/share/help 40M
/usr/share/vim 32M
/usr/share/ibus 40M
/usr/share/doc 41M
/usr/share/backgrounds 37M
/usr/share/qt5 13M
/usr/share/sounds 16M
/usr/share/midi 33M
/usr/share/locale 101M
/usr/share/fonts 163M
/usr/share/perl 21M
/usr/share/icons 96M
/usr/share/poppler 12M
/usr/share/themes 21M

/var 105.7M
/var/cache 34.4M
/var/lib 71.2M
```
按字节降序排列文件或文件夹大小
> du -s * | sort -rn

```
/usr/lib 2.1GB
920684	aarch64-linux-gnu
278532	libreoffice
236748	chromium-browser
178512	python3
156684	thunderbird
68280	gcc
45804	snapd
35288	debug
28712	python2.7
22016	python3.6
19528	git-core

/usr/share 802.8M
166684	fonts
102972	locale
97464	icons
41608	doc
40920	ibus
40276	help
37360	backgrounds
32852	midi
31768	vim
21036	perl
20628	themes
15708	sounds
14256	libreoffice
13600	i18n
13036	example-content
13016	qt5
12252	poppler

```

# 挂载磁盘不成功显示mount: /mnt: wrong fs type, bad option, bad superblock
```
#mount /dev/sdb2 /mnt
mount: /mnt: wrong fs type, bad option, bad superblock on /dev/sdb2, missing codepage or helper program, or other error.

输入 lsblk  -f 查看 /dev/sdb2 有没有文件系统格式

#lsblk -f
NAME   FSTYPE  LABEL                      UUID                                 MOUNTPOINT
sda                                                                           
├─sda1 ext4                                 01cc8123-a156-44ee-9552-9419665cf69a /boot
├─sda2 xfs                                   62121852-d67e-4a7b-b626-198c0c01f77f /
├─sda3 swap                               876fcdf2-1a4e-4318-89ed-60a9432ddc35 [SWAP]
├─sda4                                                                        
└─sda5 xfs                                   27feb26d-369a-4884-8a81-820a86414e33 /data
sdb                                                                           
├─sdb1                                                                        
├─sdb2                                          
└─sdb3                                                                        
sr0    iso9660 CentOS-8-1-1911-x86_64-dvd 2020-01-03-21-42-40-00  

可以看到 sdb2 并没有文件系统格式

输入 mkfs  -t  ext2  /dev/sdb2 格式化磁盘  

输入 mount  /dev/sdb2  /mnt

#df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        887M     0  887M   0% /dev
tmpfs           904M     0  904M   0% /dev/shm
tmpfs           904M  9.4M  894M   2% /run
tmpfs           904M     0  904M   0% /sys/fs/cgroup
/dev/sda2        50G  4.4G   46G   9% /
/dev/sda5        30G  247M   30G   1% /data
/dev/sda1       2.0G  143M  1.7G   8% /boot
tmpfs           181M  1.2M  180M   1% /run/user/42
tmpfs           181M  4.0K  181M   1% /run/user/0
/dev/sdb2        92M  1.6M   86M   2% /mnt

可以看到 sdb2 已经挂载磁盘了        
```