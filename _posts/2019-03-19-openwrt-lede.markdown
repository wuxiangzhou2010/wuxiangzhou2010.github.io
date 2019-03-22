---
layout: post
title: "OpenWRT/LEDE"
date: 2019-03-19 22:24 +0800
categories: tools
published: true
---

LEDE 是 [OpenWRT](https://Openwrt.org/) 的一个分支。 一部分 OpenWRT 开发者因为不满 OpenWRT 社区的协作方式，启动了 LEDE 项目， 后来这两支又重新合并， 并发布了新的 OpenWRT. 最新版本为 `18.06.02`. 我实际正在使用是 `WNDR4300` (WNDR2000v4 已经淘汰)

- ~~Netgear WNDR2000v4~~

  [~~`releases/18.06.2/targets/ar71xx/tiny/`~~](https://downloads.openwrt.org/releases/18.06.2/targets/ar71xx/tiny/)

- WNDR4300 v1: 属于 Nand 的 flash

  [`releases/18.06.2/targets/ar71xx/nand/`](https://downloads.openwrt.org/releases/18.06.2/targets/ar71xx/nand/)

### 刷入固件

note: 注意替换 `192.16.1.1` 为路由器 LAN 侧实际 IP

```sh
cp lede-ar71xx-tl-wr1043nd-v1-squashfs-sysupgrade.bin root@192.168.1.1:/tmp

sysupgrade -v /tmp/*.bin
# You can add the `-n` option if you DO NOT want to preserve any old configuration files and configure upgraded device from clean state (network/system settings will be lost as well)
```

- 修改 LAN 侧的 IP 地址

  note: 注意 LAN 侧的 IP 要和 WAN 侧的不要在一个网段。

```sh
uci set network.lan.ipaddr=192.168.5.1
uci commit
/etc/init.d/network restart
```

### 软件安装

- Wireguard VPN （待完善）

  ```sh
  opkg install wireguard
  opkg install luci-app-wireguard
  ```

### 设置

- [Network interfaces](https://openwrt.org/docs/guide-developer/networking/network.interfaces)

  - Physical Network Interfaces: eth0, eth8, radio0
  - Virtual Network Interfaces:eth4.0, eth4.1,br0, br-lan

- VLAN（待完善）

- Firewall zone （待完善）

  - [openwrt-configoverview](http://www.farwire.net/openwrt-configoverview.htm)
  - [openwrt-luci-proto-wireguard](https://danrl.com/blog/2016/openwrt-luci-proto-wireguard/)

### 包管理器

- [opkg](https://openwrt.org/docs/guide-user/additional-software/opkg)

  ```sh
  opkg update [your package]
  opkg install [your package]
  opkg list-upgradable
  opkg remove [your package]
  ```

- [Entware](https://github.com/Entware/Entware)

### 源码分析与编译

- [OpenWrt Development Guide](http://www.ccs.neu.edu/home/noubir/Courses/CS6710/S12/material/OpenWrt_Dev_Tutorial.pdf)

- [OpenWrt 的主 Makefile 工作过程](http://www.right.com.cn/forum/thread-73443-1-3.html) ： openwrt: Makefile 框架分析 - sammei - 博客园

- [build from source](https://openwrt.org/docs/guide-developer/source-code/start)

```sh
sudo apt update && sudo apt install git build-essential libncurses5-dev unzip python-dev -y

git clone https://github.com/openwrt/openwrt.git
cd openwrt
git checkout openwrt-18.06
```

- [Updating Feeds](https://openwrt.org/docs/guide-developer/feeds?s[]=feed)

a feed is a collection of packages which share a common location. Feeds may reside on a remote server, in a version control system, on the local filesystem.(Feeds are retrieved and managed by `scripts/feeds`, a perl script, default location: `feeds.conf.default`)

- packages
- luci
- routing
- telephony

```sh
./scripts/feeds update -a
./scripts/feeds install -a
```

`update means`: pull the latest updates for the feeds.

`install means`: making package available in make menuconfig, Only after installation will they be available in the configuration interface!

note : do not build as a `super user`

- /config : configuration files for menuconfig
- /include : makefile configuration files
- /package : packages makefile and configuration
- /scripts : miscellaneous scripts used throughout the build process
- /target : makefile and configuration for building imagebuilder, kernel, sdk and the toolchain built by buildroot.
- /toolchain : makefile and configuration for building the toolchain
- /tools : miscellaneous tools used throughout the build process

```sh
make menuconfig
# select platform and target model network feature...
make V=s -j1
```

- find the image in `openwrt/bin/targets/ar71xx/nand`
- find the package in `openwrt/bin/packages/mips_24kc`

- [[build use image builder]](https://openwrt.org/docs/guide-user/additional-software/imagebuilder)

### 问题解决

- WNDR4300 5G 丢失问题解决

  最可能 未检测到 pci 5G 硬件设备， 断电重启即可

  参考： [wndr3700v44300v1_5g_solution](http://everun.top/helpcenter/others/wndr3700v44300v1_5g_solution.html)

- 软件源问题： 更新 DNS 服务器

  参考：[openwrt 软件源配置和问题解决](https://blog.csdn.net/u010871058/article/details/77919615)

  ```
  Collected errors:
  * opkg_conf_load: Could not lock /var/lock/opkg.lock: Resource temporarily unavailable.

  解决方法：

    echo "nameserver 114.114.114.114">/tmp/resolv.conf
    rm -f /var/lock/opkg.lock
    opkg update
  ```
