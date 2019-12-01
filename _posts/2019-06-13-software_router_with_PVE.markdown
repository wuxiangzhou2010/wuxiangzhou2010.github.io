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

- 将Proxmox 写入USB

  - [etcher](https://www.balena.io/etcher/)

- 登录虚拟机软件，创建虚拟机
- 下载 koolshare LEDE
  - [koolshare LEDE](https://firmware.koolshare.cn/LEDE_X64_fw867/%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BD%AC%E7%9B%98%E6%88%96PE%E4%B8%8B%E5%86%99%E7%9B%98%E4%B8%93%E7%94%A8/)
- 将Koolshare LEDE镜像传入虚拟机
- 使用`img2kvm` 将img 转换格式转换为虚拟机支持的kvm格式

  - 转换工具[img2kvm](http://dl.everun.top/softwares/utilities/img2kvm/img2kvm)

- 下载v2ray 插件
  - [v2ray 插件](https://github.com/hq450/fancyss_history_package/tree/master/fancyss_X64)

--  [源码](https://github.com/hq450/fancysss)

## 配置  configuration

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
# create a blank vm
# downloaad lede
wget https://firmware.koolshare.cn/LEDE_X64_fw867/%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BD%AC%E7%9B%98%E6%88%96PE%E4%B8%8B%E5%86%99%E7%9B%98%E4%B8%93%E7%94%A8/openwrt-koolshare-mod-v2.31-r10822-50aa0525d1-x86-64-combined-squashfs.img.g
# downloadd img2kvm
wget http://dl.everun.top/softwares/utilities/img2kvm/img2kvm
chmod +x img2kvm
./img2kvm openwrt-koolshare-mod-v2.31-r10822-50aa0525d1-x86-64-combined-squashfs.img 100
# then add the disk to a blank vm
# default login password is "koolshare"
```
