---
layout: post
title: "Netgear WNR2000V4 flash OpenWRT"
date: 2016-05-08 12:25 +0800
categories: tools
published: false
---

上大学刚开始接触网络，印象最深的是在大一下学期，学院教务处终于允许使用电脑了。下午下课饭后， 男生宿舍那一帮去网管会的地方注册入网，８个人每人提着一台笔记本电脑并排走在路上，个个人高马大的相当壮观。

其实去注册拿电脑的目的就是检查一下每台电脑的ＭＡＣ地址，ipconfig(哈哈)。宿舍网络每个月１０元，还是相当便宜的。学校网络需要华三的 802.1x 认证，当时分新客户端和旧客户端，旧的在 win7 上不好用。并且禁止使用无线路由器。学校网络连接了教育网 CERNET，并且出口还相当大， 上下行 100M。再后来大家就习惯了在 ipv6 上下载， 网速基本能跑满。如北邮人/六维空间/晨光。软件园校区固网/无线网络都不要钱，真是方便多了。

第一次接触刷路由器还是在大四上学期的时候，偶然听说网管会的那帮会刷路由器并且支持验证和 ipv6。　我们平时能买到的路由器基本都是不支持 ipv6 的（推测其实是硬件厂商故意区分产品档次， 另一方面就是其实是自己缺钱喽，只能买低端的 Tp-link 了）。

大学时候科学上网用的都是修改 host/Google app engine/open VPN 一类的软件，换来换去了几年，一直不是很稳定。去年初偶然接触到了影梭（Shadowsocks），太ＴＭ好用了。后来发现还能刷在路由器上，简直逆天了。由于工作忙， 消退了一些热情。 去年夏天的时候，尝试移植到了树莓派１(Raspberry )上去, 但是延迟和速度并没不理想。今年四月份终于有了一段空余时间，看网上说 netgear 的几款路由器可玩性比较高就从咸鱼上搜了一个二手的路由器，Netgear WNR2000V4 的，25 元，相当划算，前前后后花了５天的时间，走了不少弯路，但是最终还是把这个 OpenWRT 给刷起来了。

| version/model      | CPU            | RAM | Flash | Network Interface |
| ------------------ | -------------- | --- | ----- | ----------------- |
| netgear wndr2000v4 | Atheros AR9341 | 32M | 4M    | 4+1 100Mbit/s     |

## 工具

- 烙铁
- 焊锡
- USB 转串口线
- Ubuntu serial 工具 Minicom

## U-boot 下载

- breed u-boot [固件地址](https://breed.hackpascal.net/EOL/)
- 选择 `breed-ar9341-wnr2000v4-r1163.bin`

  参考 [U-Boot 刷机方法大全](http://www.right.com.cn/forum/thread-154561-1-1.html)

最主要的是下面的两个命令刷入 bootloader， 虽然教程上是对 Tp-Link 的 ， 但是在 netgear 上也好用

```sh
erase 0x9f000000 +0x20000
cp.b 0x80000000 0x9f000000 0x200002
```

- 进入 U-Boot 控制台：

路由器上只在 LAN 口上接入网线，且只能有一根网线接入路由，按住路由上的复位键或 WPS/QSS 按键开机。直到所有 LED 都快速闪烁（4Hz）后，用浏览器访问 192.168.1.1 即可。

- 固件编译

  由于这个路由的 flash 计比较小， 基本不能安装什么固件， 可以在 OpenWRT 的编译选项中自行裁剪， 尝试编译固件。

参考：

- [AR/QCA/MTK Breed，功能强大的多线程 Bootloader](http://www.right.com.cn/forum/thread-161906-1-1.html)
