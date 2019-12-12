---
layout: post
title: "Wall proxy - Shadowsocks & V2ray"
date: 2019-03-11 18:52 +0800
categories: proxy
---

本文讲述搭建梯子的一般步骤

## 一台海外的虚拟主机（VPS）

- 可免费试用的 vps:

  有不少厂商提供一年的免费试用期， 比如亚马逊 AWS, 微软 Azure , 谷歌的 Google cloud platform。

  - [Amazon web services free trial](https://aws.amazon.com/free/)

    - 如亚马逊免费试用的方法, [亚马逊 AWS EC2 免费 1 年申请教程][aws free trial]
    - 注意: 超出试用限制会收费， 如流量超出 15G 会收费。

  - [Google cloud platform free trial](https://cloud.google.com/free/)
  - [Microsoft azure free trial](https://azure.microsoft.com/free/)

- 收费 vps:

  - [Bandwagon][bandwagon vps]
  - [Vultr][vultr vps]
  - [Digital ocean][digital ocean vps]

  选择配置最低的足够了，一个月 5 美元左右。

- 地理位置, 线路:

  - 根据物理规律， 地理位置越近的当然理论延迟更小。 如台湾， 日本， 新加坡。
  - 但是也要注意， 线路需要综合对比。
    - 路由绕道：去东京的线路有可能绕道美国。排除
    - 丢包率： 某些线路虽然近，但是丢包率感人。
    - 美国西部和欧洲的服务器虽然延迟大，但是可能很稳定。如 vultr 伦敦， 阿姆斯特丹。
  - 推荐 搬瓦工 [Bandwagon CN2 GIA 线路][bandwagon vps]

- 远程登陆工具:

具体登陆方法请自行百度

- Windows
  - [putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
  - [xshell](https://www.netsarang.com/en/xshell/)
  - [mobaxterm](https://mobaxterm.mobatek.net/)
- MacOs, Linux 自带即可

## 代理工具使用

### Shadowsocks

- 服务器端下载源码

  ```sh
  git clone https://github.com/shadowsocks/shadowsocks.git ss
  cd ss
  git checkout origin/master
  ```

- 配置服务器

  1. 查看服务器 IP

     ```sh
     ip addr
     ```

     注意：有些厂家的 VPS 公网 IP 可能需要在 web 管理界面查看

  2. 用上面的公网 ip 替换配置文件中的`xx.xx.xx.xx`,生成配置文件

     ```sh
     cat>config.json<<EOF
     {
        "server":"xx.xx.xx.xx",
        "server_port":8838,
        "local_port":1080,
        "password":"xx",
        "timeout":600,
        "method":"AES-256-CFB"
     }
     EOF
     ```

  3. 如果防火墙默认开启的， 需要在 web 界面配置，确保服务器相应端口开放。

- 运行服务器

  ```sh
  python shadowsocks/server.py -c config.json -d start
  ```

- 运行客户端

  Windows 下使用图形界面的客户端

  - [点击下载](https://github.com/shadowsocks/shadowsocks-windows/releases)

  需要配置的内容如图

  1. Server address
  2. Server port (同服务器配置中的 server_port)
  3. Password
  4. Encryption
  5. Proxy port (同服务器配置中的 local_port)

  ![如图](/asserts/ssconfig.png)

### [v2ray](http://v2ray.com/)

- [安装](https://v2ray.com/chapter_00/install.html)

  登录机器后以 root 的身份， 直接运行下面的脚本就可以了

  ```sh
  bash <(curl -L -s https://install.direct/go.sh)
  ```

- [设置](https://v2ray.com/chapter_00/start.html)

  1. 配置文件在 `/etc/v2ray/config.json`
  2. 其中的 users id 可以用 [这个工具](https://www.uuidgenerator.net/)链接生成
  3. 确保指定的端口开放
  4. 重启服务以生效

     ```sh
     service v2ray start
     ```

  5. [各平台客户端](https://v2ray.com/awesome/tools.html)

     客户端配置:

     ```json
     {
       "inbounds": [
         {
           "port": 1080, // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
           "listen": "127.0.0.1",
           "protocol": "socks",
           "settings": {
             "udp": true
           }
         }
       ],
       "outbounds": [
         {
           "protocol": "vmess",
           "settings": {
             "vnext": [
               {
                 "address": "server", // 服务器地址，请修改为你自己的服务器 ip 或域名
                 "port": 10086, // 服务器端口
                 "users": [{ "id": "b831381d-6324-4d53-ad4f-8cda48b30811" }]
               }
             ]
           }
         },
         {
           "protocol": "freedom",
           "tag": "direct",
           "settings": {}
         }
       ],
       "routing": {
         "domainStrategy": "IPOnDemand",
         "rules": [
           {
             "type": "field",
             "ip": ["geoip:private"],
             "outboundTag": "direct"
           }
         ]
       }
     }
     ```

     服务器端配置:

     ```json
     {
       "inbounds": [
         {
           "port": 10086, // 服务器监听端口，必须和上面的一样
           "protocol": "vmess",
           "settings": {
             "clients": [{ "id": "b831381d-6324-4d53-ad4f-8cda48b30811" }]
           }
         }
       ],
       "outbounds": [
         {
           "protocol": "freedom",
           "settings": {}
         }
       ]
     }
     ```

### [Wireguard](https://www.wireguard.com/)

Wireguard is a VPN solution in the kernel space, it has not been merged into linux source tree yet. But it is increasingly popular. We take Ubuntu for example.

- Install package

  ```sh
  sudo apt-get install software-properties-common  -y \
  && sudo add-apt-repository ppa:wireguard/wireguard -y \
  && sudo apt update \
  && sudo apt install wireguard -y
  ```

- Config

  Both server and client config need `private key` and `public key`

  ```sh
  umask 077
  wg genkey | tee privatekey | wg pubkey > publickey
  ```

  - server

    - server must enable kernel `ip_forward` feature

      ```sh
      sudo vi /etc/sysctl.conf
      # uncomment net.ipv4.ip_forward = 1
      # apply with
      sudo sysctl -p
      ```

    - create server config file

      ```sh
      sudo vi /etc/wireguard/wg0server.conf
      ```

    - server config is as follows:

      ```sh
      [Interface]
      Address = 10.0.0.1/24  # This is the vpn address, remember to change eth0 to the interface of you server
      PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
      PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
      ListenPort = 51820
      PrivateKey = [SERVER PRIVATE KEY]

      [Peer]
      PublicKey = [CLIENT PUBLIC KEY]
      AllowedIPs = 10.0.0.2/32  # This denotes the clients IP with a /32: the client only has ONE IP.
      ```

      - start/stop service

      ```sh
      sudo wg-quick up wg0server
      sudo wg-quick down wg0server
      # start when reboot
      sudo systemctl enable wg-quick@wg0server
      ```

  - client

    - create client config file

      ```sh
      sudo vi /etc/wireguard/wg0.conf
      ```

    - client config is as folows:

      ```sh
      [Interface]
      Address = 10.0.0.2/24  # The client IP from wg0server.conf with the same subnet mask
      PrivateKey = [CLIENT PRIVATE KEY]
      DNS = 10.0.0.1 # the server need to enable dns service

      [Peer]
      PublicKey = [SERVER PUBLICKEY]
      AllowedIPs = 0.0.0.0/0, ::0/0
      Endpoint = [Server ip address]:51820 # server ip or domain name
      PersistentKeepalive = 25
      ```

    - Both IOS and MacOs has working official app

refer to：

- [Arch linux wiki: wireguard](https://wiki.archlinux.org/index.php/WireGuard)

- [linode: set-up-wireguard-vpn-on-ubuntu](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/#wireguard-client)
  [Wireguard VPN: Typical Setup](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/)

[bandwagon vps]: https://bandwagonhost.com/aff.php?aff=55556
[vultr vps]: https://www.vultr.com/
[digital ocean vps]: https://www.digitalocean.com/
[aws free trial]: https://www.beizigen.com/2090.html
