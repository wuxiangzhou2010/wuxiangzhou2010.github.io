---
layout: post
title: "Software router with PVE"
date: 2019-06-13 12:12 +0800
categories: tools
published: false
---

## 安装步骤

- 下载 Proxmox 虚拟机软件 ISO

  - [Proxmox VE](https://www.proxmox.com/en/downloads)

- 将 Proxmox 写入 USB

  - [etcher](https://www.balena.io/etcher/)

- 登录虚拟机软件，创建虚拟机

  - 不创建磁盘
  - 绑定所有网卡，注意网卡标号与实际物理接口对应关系

- 下载 koolshare LEDE
  - [koolshare LEDE](https://firmware.koolshare.cn/LEDE_X64_fw867/%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BD%AC%E7%9B%98%E6%88%96PE%E4%B8%8B%E5%86%99%E7%9B%98%E4%B8%93%E7%94%A8/)
- 使用`scp` 将 Koolshare LEDE 镜像传入虚拟机
- 使用`img2kvm` 将 img 转换格式转换为虚拟机支持的 kvm 格式

  - 转换工具下载地址[img2kvm](http://dl.everun.top/softwares/utilities/img2kvm/img2kvm)

- 下载 v2ray 插件
  - [v2ray 插件](https://github.com/hq450/fancyss_history_package/tree/master/fancyss_X64)
  - [插件源码](https://github.com/hq450/fancysss)

## 配置 configuration

```json
{
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "xxxxx",
          "port": xxxxx,
          "users": [
            {
              "id": "xxxxxx",
              "alterId": 10
            }
          ]
        }
      ]
    }
  }
}
```

## 相关命令

```sh
# 1. 创建一个空虚拟机， 不添加磁盘
# 2. 下载 OpenWRT
wget https://firmware.koolshare.cn/LEDE_X64_fw867/%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BD%AC%E7%9B%98%E6%88%96PE%E4%B8%8B%E5%86%99%E7%9B%98%E4%B8%93%E7%94%A8/openwrt-koolshare-mod-v2.31-r10822-50aa0525d1-x86-64-combined-squashfs.img.g

# 3. 下载 img2kvm
wget http://dl.everun.top/softwares/utilities/img2kvm/img2kvm
# 4. 添加执行权限
chmod +x img2kvm
# 5. 转换磁盘格式， 其中100 为新建的虚拟机的编号
./img2kvm openwrt-koolshare-mod-v2.31-r10822-50aa0525d1-x86-64-combined-squashfs.img 100

# 6. hen add the disk to a blank vm
# 7. default login password is "koolshare"
```
