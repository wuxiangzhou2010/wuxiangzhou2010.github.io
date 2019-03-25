---
layout: post
title: "Wall proxy - Shadowsocks & V2ray"
date: 2019-03-11 18:52 +0800
categories: proxy
---

1. 一台海外的虚拟主机（VPS）:

   - 免费 vps:

     有不少厂商提供一年的免费试用期， 比如亚马逊 aws, 微软 azure , 谷歌的 google cloud platform。

     - [Amazon web services free trial](https://aws.amazon.com/free/)

       如亚马逊免费使用的方法, [亚马逊 AWS EC2 免费 1 年申请教程][aws free trial]

     - [Google cloud platform free trial](https://cloud.google.com/free/)
     - [Microsoft azure free trial](https://azure.microsoft.com/free/)

   - 收费 vps:

     如 [bandwagon][bandwagon vps], [vultr][vultr vps], [digital ocean][digital ocean vps] 的话，选择配置最低的足够了，一个月 5 美元左右。

   - 地理位置:

     尽量选择近一点的， 如台湾， 日本， 新加坡。 但是也要注意延迟和丢包率。美国西部和欧洲的服务器虽然延迟大，但是可能更稳定。

2. 远程登陆工具:

   推荐 [putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)， [xshell](https://www.netsarang.com/en/xshell/), 用来登录购买的机器, 安装,设置,并启动代理软件

3. 代理工具:

   推荐 `v2ray` 或者 `shadowsocks`

   基本使用方法参考如下

- Shadowsocks

  使用上面的 ssh 工具登录服务器

  - 服务器端下载源码

    ```sh
    git clone https://github.com/shadowsocks/shadowsocks.git ss
    cd ss
    git checkout origin/master
    ```

    若提示 git 未安装， 使用一下命令安装

    ```sh
    sudo apt install git -y
    ```

  - 配置服务器

    1. 查看服务器 IP， 并替换下面的配置文件中的`xx`

       ```sh
       ip addr
       ```

    2. 生成服务端配置文件

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

    3. 确保服务器相应端口开放。

  - 运行服务器

    ```sh
    python shadowsocks/server.py -c config.json -d start
    ```

  - 运行客户端

    windows 下使用图形界面的客户端， [点击下载](https://github.com/shadowsocks/shadowsocks-windows/releases)

    需要配置的内容如图

    1. server address
    2. server port (同服务器配置中的 server_port)
    3. password
    4. encription
    5. proxy port (同服务器配置中的 local_port)

    ![如图](/asserts/ssconfig.png)

- [v2ray](http://v2ray.com/)

  - [安装](https://v2ray.com/chapter_00/install.html)

    登录机器后以 root 的身份， 直接运行下面的脚本就可以了

    ```sh
    bash <(curl -L -s https://install.direct/go.sh)
    ```

  - [设置](https://v2ray.com/chapter_00/start.html)

    1. 配置文件在 `/etc/v2ray/config.json`
    2. 其中的 users id 可以用 [这个](https://www.uuidgenerator.net/)链接生成
    3. 确保指定的端口开放
    4. 重启服务以生效

       ```sh
       service v2ray start
       ```

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

- [Wireguard](https://www.wireguard.com/)

  Wireguard is a VPN solution in the kernel space, it has not been merged into linux source tree yet. But it is increasingly popular. We take ubuntu for example.

  - Install package

    ```sh
    sudo add-apt-repository ppa:wireguard/wireguard -y \
    && sudo apt update \
    && sudo apt install wireguard-dkms wireguard-tools -y
    ```

  - Config

    both server and client config need `private key` and `public key`

    ```sh
    umask 077
    wg genkey | tee privatekey | wg pubkey > publickey
    ```

    - server

    server need to enable kernel `ip_forward` feature

    ```sh
    sudo vi /etc/sysctl.conf
    # uncomment net.ipv4.ip_forward = 1
    # apply with
    sudo sysctl -p
    ```

    create server config file

    ```sh
    sudo vi /etc/wireguard/wg0server.conf
    ```

    server config as follows:

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

    start

    ```sh
    sudo wg-quick up/down wg0server
    # start when reboot
    sudo systemctl enable wg-quick@wg0server
    ```

    - client

    create client config file

    ```sh
    sudo vi /etc/wireguard/wg0.conf
    ```

    client config as folows:

    ```sh
    [Interface]
    Address = 10.0.0.2/24  # The client IP from wg0server.conf with the same subnet mask
    PrivateKey = [CLIENT PRIVATE KEY]
    DNS = 10.0.0.1

    [Peer]
    PublicKey = [SERVER PUBLICKEY]
    AllowedIPs = 0.0.0.0/0, ::0/0
    Endpoint = [Server ip address]:51820 # this is the server real IP
    PersistentKeepalive = 25
    ```

    ```sh
    sudo wg-quick up/down wg0
    # enable start on system boot
    sudo systemctl enable wg-quick@wg0
    ```

refer to：

[Arch linux wiki: wireguard](https://wiki.archlinux.org/index.php/WireGuard)

[linode: set-up-wireguard-vpn-on-ubuntu](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/#wireguard-client)

[bandwagon vps]: https://bandwagonhost.com/
[vultr vps]: https://www.vultr.com/
[digital ocean vps]: https://www.digitalocean.com/
[aws free trial]: https://www.beizigen.com/2090.html
