---
layout: post
title: "Enable Goolge BBR feature"
date: 2019-03-13 17:42 +0800
categories: tcp_ip
published: true
---

BBR (Bottleneck Bandwidth and Round-trip propagation time)is contributed by Google to the linux kernel tree, it is added `from 4.9.0` kernel but is not enabled by default. To enable it, we take Ubuntu for example.

```sh
# add  BBR module to kernel
sudo modprobe tcp_bbr

# Add to default loaded modules
sudo echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
sudo echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
sudo echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

# apply with
sudo sysctl -p

# check
sudo sysctl net.ipv4.tcp_available_congestion_control
sudo sysctl net.ipv4.tcp_congestion_control

```

For more about

- [modprobe,rmmod,lsmod](https://linux.die.net/man/8/modprobe)

reference:

- [TCP BBR congestion control comes to GCP – your Internet just got faste](https://cloud.google.com/blog/products/gcp/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster)
