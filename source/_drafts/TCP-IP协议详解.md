---
title: TCP-IP协议详解
tags: 网络
categories: 笔记
---

<h4>章节一</h4>
1. IP地址分为5类

2. 域名系统DNS

3. TCP传给IP的数据单元，叫做TCP报文(TCP segment)，IP传给网络接口层的数据单元，叫做IP数据报(IP datagram)，通过以太网的数据流，被称为帧（Frame）。

4. 以太网数据帧的物理特性是其长度，必须在46 ~ 1500字节之间。

5. IP跟网络接口层之间传输的数据单元是packet，packet可以是IP数据报，也可以是IP数据报的一个片（fragment）。

6. 知名端口号：FTP(21)，Telnet(23)，TFTP(69)，Rlogin(513)

7.  802.3规定IP数据报长度为38 - 1492，以太网帧则未定为46 - 1500.

8. SLIP定义的帧格式，在首尾都会传一个0xc0，报文中如果遇到0xc0，会用0xdb0xdc来代替0xc0,0xdb被称为ESC，如果遇到0xdb，需要用0xdb0xdd来代替。

9. 
