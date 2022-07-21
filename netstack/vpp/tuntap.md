# VPP TUNTAP支持

TUNTAP作为Linux系统上实现用户态网络应用的典型接口，可以实现两个关键的功能：


  * 实现从用户态程序构造网络数据包后注入内核网络协议栈
  * 通过配置内核网络（如路由）实现从内核网络接受数据包

正是因为这样的能力，TUNTAP接口成为大量L2/L3 VPN技术实现的基石。

VPP做为功能丰富的用户态协议栈，在较早的版本中就引入了TUNTAP的支持，并在较新的版本引入了virtio能力的支持。

在最新的版本上VPP上创建TAP接口非常简单，可以采用vppctl create tap命令即可创建一个tap接口，并使用vpp丰富的功能为该接口配置IP及路由。

除了基本的与内核相连外，还具备如下非常有用的能力：

  * 多队列支持，方便利用多cpu实现较高性能
  * 指定内核侧namespace能力，方便实现网络隔离
  * 指定内核侧接口IP地址/路由的能力
  * checksum offload能力


# 相关链接

  1. [Linux内核官方TUNTAP文档](https://www.kernel.org/doc/Documentation/networking/tuntap.txt)
