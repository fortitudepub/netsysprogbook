# 什么是kernel bypass

由于本书主要处理网络相关技术，这里的kernel bypass即指在网络技术这个特定范畴中的kernel bypass。

kernel bypass从字面上来说，就是bypass kernel，结合linux环境以及网络技术这个范畴，即跳过Linux内核的网络处理逻辑。那么认证来承载
网络处理逻辑呢？这便是各种kernel bypass技术的作用之处了。

当前主流的kernel bypass技术则是DPDK这个由intel所开源的技术，除外还有netmap等开源或私有的技术，由于流行度不如DPDK，不是我们这里所要关注的重点了。

除外，近年来，因为容器技术的广泛使用，基于EBPF的XDP这种kernel bypass技术也较为流行，本书也将补充相关知识。
