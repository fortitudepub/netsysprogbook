# 简介

对于传统的二三层网管交换机而言，每个交换机端口可以配置成access/trunk模式，前者只能接入特定的VLAN（过来的帧是untagged的普通Ethernet帧），后者
则可以允许多个VLAN的以太帧（即802.1Q封装），在交换机上，则可以虚拟出多个VLAN，对于三层交换机，还可以在每个VLAN上创建三层VLAN虚拟接口，负责VLAN
内接入主机的DHCP、DNS等能力；并负责VLAN间的路由转发。

# VPP

VPP的交换能力与普通交换机不同，这是因为VPP的定位是一台高性能的路由器，路由器与交换机的典型区别即接口没有access/trunk这样的模式之分。

但VPP实际上还是具备简单的bridge能力的，并可以创建bridge domain以及bvi接口，但目前我所使用的，只是将一个接口直接插入到bridge中，那这个bridge
里任一接口的广播报文都会送给其他接口，且没有用到任何VLAN能力，对于802.1Q报文的处理也是未知状态。

因此，我们这里主要调研其VLAN支持能力，也即回答如下核心问题：

  * VPP能否支持类似于access能力，即与接入VLAN相同的trunk/access接口形成一个bridge domain
  * VPP能否支持类似于trunk能力，即与接入VLAN相同的trunk/access接口形成一个bridge domain


## 验证

先创建两个接口连接宿主Linux机器，以便在宿主Linux机器上创建VLAN子接口，模式接入VPP的二层客户端。


```
vppctl create tap id 100 host-if-name tap100
vppctl create tap id 200 host-if-name tap200
vppctl set interface state tap100 up
vppctl set interface state tap200 up

```

在宿主机tap100接口上创建一个VLAN id为100的子接口，并配置IP地址，通过ping同网段地址产生广播报文。

```
ip link add link tap100 name tap100.100 type vlan id 100
ip link set tap100 up
ip link set tap100.100 up
ip addr add 192.168.100.2/24 dev tap100.100
```

此时VPP上trace结果如下，说明VPP的tap100接口默认是不处理VLAN报文的。

```
00:20:30:464701: virtio-input
  virtio: hw_if_index 1 next-index 4 vring 0 len 46
    hdr: flags 0x00 gso_type 0x00 hdr_len 0 gso_size 0 csum_start 0 csum_offset 0 num_buffers 1
00:20:30:464711: ethernet-input
  ARP: 02:fe:cf:22:52:bb -> ff:ff:ff:ff:ff:ff 802.1q vlan 100
00:20:30:464718: error-drop
  rx:tap100
00:20:30:464723: drop
  ethernet-input: unknown vlan
```

经过查阅VPP的资料，可以用如下方式实现VLAN以及接口以trunk方式接入VLAN。

```
create bridge-domain 100
create sub-interface tap100 100 dot1q 100
create sub-interface tap200 100 dot1q 100
set interface l2 bridge tap100.100 100
set interface l2 bridge tap200.100 100
create loopback interface
set interface l2 bridge loop0 bvi
set interface l2 tag-rewrite loop1 push dot1q 100
```

此方式解释如下：
  * 接口仍以子接口方式接入bridge，这样即可接收打上特定VLAN的报文。
  * bvi虚拟接口，以access方式接入bridge，因此需要配置tag-rewrite，VPP会在l2-input feature中打开l2-input-vtr节点，给bvi接口发出的报文打上VLAN 100，同时也意味着接收的tag 100的报文会被移除。

此方式可实现如下功能：
  * 宿主机上的tap100/tap200接入的相同VLAN之间可以通过VPP进行交换。
  * 宿主机上的tap100/tap200接入的相同VLAN的接口，可与VPP bvi进行双向通信。


## 总结

从上面的验证可以看出，实际上VPP是具备较为完整的VLAN交换能力的，但是与传统的二三层交换机不太一样，需要借助于子接口（sub-interface）这个对象来配置VLAN，而非传统交换机的switch port access/trunk allow
等方式。

# 参考资料
 1. [Wiki上的802.1Q介绍](https://en.wikipedia.org/wiki/IEEE_802.1Q)
