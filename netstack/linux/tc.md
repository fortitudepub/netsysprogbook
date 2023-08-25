# TC

Linux TC原为traffic control，主要用于进行报文的QoS处理；现如今已经泛化为内核中对数据包进行各种复杂
操作的工具箱了，甚至ebpf这样的程序，也可以借助于tc加载进内核。

由于TC操作较为复杂，以下记录各种我曾经用到的TC操作。

## TC 对报文进行编辑

```
# 添加入方向qdisc.
tc qdisc add dev wlo1 handle ffff: ingress
# 对ip地址进行修改并重做ip csum。
tc filter add dev wlo1  protocol ip parent ffff: u32 match ip dst 192.168.31.178/32 flowid :1  action pedit  munge ip dst set 100.64.0.2 pipe csum ip  pass
# 对eth dst以及ip dst进行修改，并重做csum.
tc filter add dev wlo1  protocol ip parent ffff: u32 match ip dst 192.168.31.178/32 flowid :1  action pedit ex munge eth dst set 28:df:eb:18:69:29 pipe action pedit  munge ip dst set 100.64.0.2 pipe csum ip  pass
# 对ip地址进行修改，并重做csum，涉及到pseudo header，亦需要重算传输层。
tc qdisc add dev eth0 handle ffff: ingress
tc filter add dev eth0  protocol ip parent ffff: u32 match ip dport 2288  flowid :1  action pedit  munge ip dst set 192.168.1.2 pipe csum ip and udp and tcp pass

```


