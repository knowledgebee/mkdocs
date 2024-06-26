---
title: 配置虚拟机 Vbox 网络
date: 2022-08-30 21:44:26
tags:
    - Network
---

如何使用和配置 VBox 中虚拟机的网络

<!--more-->

VBox 是一个开源的虚拟机软件，由软件公司 Sun 开发，在 Sun 被甲骨文收购后属于甲骨文。VBox 界面干净，操作简单。想要与 VBox 中的虚拟机通信需要对虚拟机的网络进行配置。VBox 中的网络有三种：网络地址转换( NAT )、仅主机( Host Only )和桥接(Bridge)。

NAT 模式：为虚拟机提供一个虚拟的网络接口并分配一个虚拟的 IP 地址和一个虚拟的网关地址，虚拟机通过该虚拟的 IP 地址和虚拟的网关与互联网上的其他主机通信，VBox 将该虚拟网卡的收到的数据包进行 NAT 之后通过主机发送到互联网，这种模式只能虚拟机发起通信，互联网中的其他网络设备无法通过该虚拟网卡访问该虚拟机，在经过 NAT 之后，网络上的其他主机会以为是在和该虚拟机的主机通信，因为在 NAT 模式下虚拟机内部看起来是使用的虚拟 IP 地址和其他主机通信，但是在进行 NAT 转换后使用的还是主机的 IP 地址。

Host Only 模式：该模式为虚拟机提供网络接口和 IP 地址，没有网关地址，无法与其他网段的主机进行通信，在创建 Host Only 网络时，VBox 会同时在主机上创建一个名为 Host Only 的虚拟网卡，该虚拟网卡需配置在 Host Only 网络的网段内，同时还需要是私网 IP 网段，否则可能影响主机网络，主机通过该虚拟网卡与 Host Only 网络通信，Host Only 网络只有该网段内可以相互通信，与其他网络相隔离，互联网中的其他网络设备无法访问该虚拟网络，仅有主机和主机中处于相同相同网络的虚拟机可以相互访问。

Bridge 模式：该模式为虚拟机提供一个虚拟的网络接口，该接口有 MAC 地址，有着和正常网口一样的功能，同时会挂载在主机的网口上，使用主机的网口通信，在有 DHCP 服务器为其分配了相同网段的 IP 时，主机和相同网络中的其他设备都可以与该虚拟机相互访问。

### **Notice**

***

\* 在配置 VBox 的网络时需要注意 Host Only 的虚拟网卡的配置，需要将此网卡以及虚拟机的 IP 段设置为私网 CIDR 段，配置子网掩码，同时尽量避开宿主机所在的局域网的 CIDR 段，以防与其他计算机的 IP 地址冲突，影响正常的网络通信。

\* 在配置 IPv6 时需要调整 NAT。
