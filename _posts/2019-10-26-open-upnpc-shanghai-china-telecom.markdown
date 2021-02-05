---
layout: post
title: "上海电信SDN网关，UPNP 端口映射"
date: 2019-10-26 12:12 +0800
categories: tools
published: true
---

最近刚搬家， 合租屋的路由没有完全控制权限， port forwarding 也就无法配置了。 偶然查找了一个工具， 可以通过 uPnP [IGDP](https://en.wikipedia.org/wiki/Internet_Gateway_Device_Protocol)(Internet Gateway Device Protocol) 添加一个端口映射， 具体使用还有待观察。

- [Tunnelling SSH via uPnP](https://medium.com/@jos.martin/tunnelling-ssh-via-upnp-af023d04290d)

## 自行编译

```sh
git clone https://github.com/miniupnp/miniupnp.git
docker run -it -v /Users/takesachishuu/miniupnp:/root ubuntu /bin/bash
```

## Mac

```sh
brew install upnpc
```

## 使用方法

```sh

Usage : ./upnpc-static [options] -a ip port external_port protocol [duration]
        Add port redirection
        ./upnpc-static [options] -d external_port protocol <remote host>
        Delete port redirection
        ./upnpc-static [options] -s
        Get Connection status
        ./upnpc-static [options] -l
        List redirections
        ./upnpc-static [options] -L
        List redirections (using GetListOfPortMappings (for IGD:2 only)
        ./upnpc-static [options] -n ip port external_port protocol [duration]
        Add (any) port redirection allowing IGD to use alternative external_port (for IGD:2 only)
        ./upnpc-static [options] -N external_port_start external_port_end protocol [manage]
        Delete range of port redirections (for IGD:2 only)
        ./upnpc-static [options] -r port1 [external_port1] protocol1 [port2 [external_port2] protocol2] [...]
        Add all redirections to the current host
        ./upnpc-static [options] -A remote_ip remote_port internal_ip internal_port protocol lease_time
        Add Pinhole (for IGD:2 only)
        ./upnpc-static [options] -U uniqueID new_lease_time
        Update Pinhole (for IGD:2 only)
        ./upnpc-static [options] -C uniqueID
        Check if Pinhole is Working (for IGD:2 only)
        ./upnpc-static [options] -K uniqueID
        Get Number of packets going through the rule (for IGD:2 only)
        ./upnpc-static [options] -D uniqueID
        Delete Pinhole (for IGD:2 only)
        ./upnpc-static [options] -S
        Get Firewall status (for IGD:2 only)
        ./upnpc-static [options] -G remote_ip remote_port internal_ip internal_port protocol
        Get Outbound Pinhole Timeout (for IGD:2 only)
        ./upnpc-static [options] -P
        Get Presentation url


# list all port mapping (not working with SDN gateway as of Dec. 2019)
upnpc -l

# delete
upnpc -d 12345 tcp
upnpc -d 2345 udp

#  add mapping internal ip, internal port, external port， protocol
upnpc -a 192.168.0.2 80 80 TCP
```

- ps： 电信的网关 SDN 网关设备对 uPnP 的支持不全面， 有些命令可能不好用。

- [python verson](https://pypi.org/project/miniupnpc)

- [A shell script](https://gist.github.com/wuxiangzhou2010/11c2d7848b4741b8887fc88bebe9277b)

## NAT traversal

One solution for NAT traversal, called the `Internet Gateway Device Protocol (IGD Protocol)`, is implemented via `UPnP`. Many routers and firewalls expose themselves as Internet Gateway Devices, allowing any local UPnP control point to perform a variety of actions, including `retrieving the external IP address of the device`, enumerate existing port mappings, and add or remove port mappings. By adding a port mapping, a UPnP controller behind the IGD can enable traversal of the IGD from an external address to an internal client.

- uPnP IGDP (Universal Plug and Play Internet Gateway Device Protocol)
- NAT-PMP ( NAT Port Mapping Protocol)
- PCP (Port Control Protocol )

## refer

- [check open ports](https://www.yougetsignal.com/tools/open-ports/)
