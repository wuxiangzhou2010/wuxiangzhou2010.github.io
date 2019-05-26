---
layout: post
title: "TCP UDP learning note"
date: 2018-05-20 11:15 +0800
categories: net
published: true
---

This is a learning note of TCP/IP illustrated volume 1, I mainly focused on TCP/UDP and the difference.

## Chapter 11. UDP: User Datagram Protocol

- 11.1 Inctoduction
- 11.2 UDP Header

the User Datagram Protocol header has four fields, each of which is 2 bytes.

- source port, destination port
- length and checksum

udp is a datagram-oriented protocol. it `does not provide error correction`, `sequencing`, `duplicate elimination`, `flow control or congestion control`.

it can provide `error detection`. the checksum field is end to end and is computed over the UDP pseudo-header, which includes the source and destination IP addresses from the IP hader, Thus, any modification(NAT) made to those fields requires a modification to the UDP checksum

each UDP output operation requested by an application produces `exactly one UDP datagram`, which cause one ip datagram to be sent.

- [FRC0768] is the official specification of UDP, and it has remained as a standard without significant revisions for more than 30 year.

- 11.3 UDP Checksum
- 11.4 A simple example
- 11.5 IP Fragmentation
  - Fragmentation can take place either at the original sending host or at an intermediate router.
  - when an IP datagram is fragmented, it is not reassembled until it reaches its final destination.
- 11.6 ICMP Unreachable(Fragmentation Required)
- 11.7 Determining the Path MTU Using Traceroute
- 11.8 Interaction Between UDP and ARP
- 11.10 Maxium UDP Datagram Size
- 11.11 ICMP Source Quench Error
- 11.12 UDP Server Design
- 11.13 Summary

## Chapter 17. TCP: transmission Control Protocol

- 17.1 Introduction
- 17.2 TCP Service
- 17.3 TCP Header

  tcp provides a connection-oriented, reliable, byte stream service

  - source port and destination port 32bits
  - sequence number 32bits
  - acknowledgement number 32bits
  - `URG|ACK|RST|SYN|FIN`, window size 32bits
  - TCP checksum urgent pointer 32bits

- 17.4 Summary

## Chapter 18. TCP Connection Establishment and Termination

- 18.1 Introduction
- 18.2 Connection Establishment and Termination
- 18.3 Timeout of Connection Establishment
- 18.4 Maximum Segment Size
- 18.5 TCP half close
- 18.6 TCP state transition diagram
- 18.7 Reset segment
- 18.8 simultaneous open
- 18.9 simultaneous close
- 18.10 TCP options
- 18.11 TCP server Design
- 18.12 Summary

## Chapterr 21. TCP Timeout and Retransmission

- 21.1 Introduction
- 21.2 Simple Timeout and Retansmission Example
- 21.3 Round-Trip Time Measurement
- 21.4 An RTT example
- 21.5 Congestion Example
- 21.6 Congestion Avoidance Algorithm
- [TCP 拥塞 控制](https://zhidao.baidu.com/question/98620785.html)

  1. 慢启动(Slow start)
  2. 拥塞避免(Congestion avoidance)
  3. 快速重传(Fast retransmit)
  4. 快速恢复(Fast Recovery)

  - 拥塞窗口(cwnd)
  - 接收窗口(rwnd)
  - 网络往返时间(Round Trip Time, RTT)

- 21.7 Fast Retransmit and Fast Recovery Algorithm
- 21.8 Congestion Example(continued)
- 21.9 Per-Route Metrics
- 21.10 ICMP Errors
- 21.11 Repacketization
- 21.12 Summary

## tpp/udp difference

1. tcp is `connection oriented`, whereas udp is connection-less.
2. This means that TCP tracks all data sent, requiring `acknowledgement` for each octet.
3. tcp is considered a `reliable` datatransfer protocal because of acknowledgement. It ensure that no data is sent to upper layer out of order, duplicated, or has missing pieces. it can even manage transmissions to reduce `congestion`.

reference:

- [book: TCP IP Illustrated Volume 1 The Protocols](https://doc.lagout.org/network/TCP%20IP%20Illustrated%20Volume%201%20The%20Protocols.pdf)

- [kernel_flow-->TCP](https://wiki.linuxfoundation.org/networking/kernel_flow)
- [path-of-packet](https://www.cs.dartmouth.edu/~sergey/me/netreads/path-of-packet/Lab9_modified.pdf)