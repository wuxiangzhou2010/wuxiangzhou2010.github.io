---
layout: post
title: "编译 Lean Openwrt"
date: 2019-12-12 12:12 +0800
categories: tools
published: true
---

看到最近网上的大 V 越来越多提到了 Lean OpenWRT, 于是也来做一下尝试。目标对象是家里的一台 x86 架构的软路由。

先放上源码仓库：

- https://github.com/coolsnowwolf/lede.git

## 准备工作

- 系统

  这里选用 Ubuntu 18.04， 最好使用一台 VPS，网速比较快。
  (我这里使用的家里 PVE 上的一台虚拟机, 放在科学上网环境之后。)

- 安装必要工具

  ```sh
  docker run --name "builder" -d -it debian:latest /bin/bash
  docker exec -it builder /bin/bash

  apt-get update -y
  apt-get -y --no-install-recommends \
  install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler git  wget \
   vim \
  bash-completion \
  ca-certificates \
  vim \
  locales-all \
  rsync

  useradd -m testuser
  echo -e "123\n123" | passwd testuser
  su testuser
  cd


  ```

- 克隆仓库, 更新软件包

  ```sh
  git clone https://github.com/coolsnowwolf/lede

  # enable helloworld
  cd lede
  #vi feeds.conf.default
  sed -i 's/^#\(.*helloworld\)/\1/' feeds.conf.default
  ./scripts/feeds update -a && ./scripts/feeds install -a
  # make menuconfig
  make defconfig
  make -j8 download
  make -j$(($(nproc) + 1)) V=s
  ```

- 选择系统, 架构和软件

  - 使用方向键跳转
  - 使用 enter 键 进入选项
  - 双击 esc 退出选项
  - 使用空格键选择
  - 使用`?`查看选项的简介
  - 使用`/`查找选项

    --> Target System 选择 `x86`
    ![openwrt-target](/asserts/openwrt-target.png)

    --> SubTarget 选择 `x64 64bit`
    ![openwrt sub target ](/asserts/openwrt-sub-target.png)
    -->Luci --> Applications
    在按需求选择自己需要的插件。
    ![luci applications](/asserts/openwrt-luci-applications.png)

## 编译

输入如下命令，-j1 后面是线程数。第一次编译推荐用单线程，国内请尽量全局科学上网

```sh
make -j1 V=s
```

![编译完成](/asserts/openwrt-complete.png)

## 固件

固件路径 `lede/bin/targets/x86/64`

```sh
cd bin/targets/x86/64
ls -l
-rw-r--r-- 1 zzz zzz       108 Dec 12 10:45 config.seed
-rw-r--r-- 1 zzz zzz 185073664 Dec 12 11:33 openwrt-x86-64-combined-squashfs.img
-rw-r--r-- 1 zzz zzz  39976960 Dec 12 11:33 openwrt-x86-64-combined-squashfs.vmdk
-rw-r--r-- 1 zzz zzz     11503 Dec 12 11:33 openwrt-x86-64-generic.manifest
-rw-r--r-- 1 zzz zzz  35851304 Dec 12 11:33 openwrt-x86-64-rootfs-squashfs.img
-rw-r--r-- 1 zzz zzz   3760288 Dec 12 11:31 openwrt-x86-64-vmlinuz
drwxr-xr-x 2 zzz zzz     20480 Dec 12 10:57 packages
-rw-r--r-- 1 zzz zzz       573 Dec 12 11:34 sha256sums
```

- `.img` 格式文件可以通过 [etcher](https://www.balena.io/etcher/) 或者 dd 直接写入磁盘， 或者通过 [img2kvm](http://everun.top/helpcenter/others/img2kvm-instruction.html) 倒入到 pve 中使用
- `·vmdk` 格式文件是 vmware 支持的格式。

## 注意

- 不要用 root 用户 git 和编译！！！
- 国内用户编译前最好准备好梯子
- 默认登陆 IP 192.168.1.1, 密码 password， 请修改密码

## 参考

- [编译 Lean 的 Openwrt 固件全攻略](https://imgki.com/archives/openwrt-lean.html)

## ps

- 可以直接选择生成 PVE 支持的镜像格式， 如[这个链接](https://lala.im/5523.html)
