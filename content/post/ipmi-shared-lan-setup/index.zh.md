---
title: "共享端口的IPMI设置"
date: 2014-03-26T22:51:49+08:00
description: "通过 VLAN 复用 eth0 端口，在共享网口上使用 IPMI"
slug: ipmi-shared-lan-setup
tags:
    - IPMI
    - 服务器
    - VLAN
draft: false
---

IPMI可以让系统管理员远程控制服务器硬件，无论操作系统处于什么状态，甚至无论是否开机，只要电源已接通，就可以接受控制，因而IPMI对于实验用途的服务器意义巨大。我们的集群之前一直使用专用的IPMI网络，这需要额外的交换机、网线投入，也需要额外的安装、维护投入，而且会使线路杂乱而不利于管理。

这次我们新配置的集群打算使用共享端口及其线路的方式来使用IPMI，其基本原理是通过VLAN复用eth0端口/线路，业务网和IPMI网分别运行在不同的VLAN当中。当eth0端口收到报文，会先判断其VLAN ID，若与IPMI的ID一致，则将报文转发给IPMI子系统，否则交给正常的报文处理逻辑。

当我兴致勃勃地进入BIOS，想给IPMI配置网络的时候，却发现只能修改IP地址、子网掩码和网关，而网络则只能是“dedicated”，该选项是灰色，不能改。如下图所示：

![BIOS 中 IPMI 网络端口只能选 dedicated](ipmi-dedicated.jpg)

我怀疑“IPMI LAN Selection”选项不可改是因为主板集成的是比较新的万兆网卡，IPMI相关功能尚不完善，于是暂时放弃。不过当晚在网上搜索资料时，无意中发现该网卡是支持“IPMI passthrough”功能的，从字面看貌似相关功能应该是完备的，于是我决定再次尝试一下。

这次我先在BIOS里设置好IPMI的静态IP地址、子网掩码和网关，然后通过IPMI专用端口连接网络，再用浏览器打开IPMI的web配置界面，在Configuration / Networking页面里找到了共享端口相关的设置，如下图所示。我将VLAN设置为“enable”，VLAN ID设置为“100”，Lan Interface设置为“Share”，然后通过eth0端口和VLAN 100成功访问到了IPMI功能。

![IPMI web 配置界面中的共享端口与 VLAN 设置](ipmi-shared.jpg)

配置好共享端口之后，再次进入BIOS就可以看到IPMI的网络端口已变成“Share LAN”，而专用网络连接的状态变成“No Connect”。

![BIOS 中显示 Share LAN 已启用](ipmi-vlan.png)
