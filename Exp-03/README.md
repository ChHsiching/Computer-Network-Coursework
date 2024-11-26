# Exp3 - RIP & OSPF 路由协议

---

> RIP路由协议的配置

## 网络拓扑结构 Network Topology 

### 1. 设备清单

- 路由器 * 2  `1841 Router`
- 交换机 * 2  `2950-24 Switch`
- 计算机 * 2  `PC-CT PC`

### 2. 设备连接

1. PC - Switch

    `直通线` 连接 `FastEthernet` 以太网口

2. Switch - Router

    `直通线` 连接 `FastEthernet` 以太网口

3. Router - Router

    `交叉线` 连接 `FastEthernet` 以太网口
    > 这里当然也可以使用 Serial 串口连接，但是我们不需要通过它进行 PPP 封装或者是 PAP、CHAP 认证，因此直接使用网口连接，简单快捷 

    由于 Cisco 思科 路由器的网口默认状态为 `shutdown`，需要手动打开

### 3. Router 路由器配置

分配各个接口的 IP 地址并打开网口

```shell
# 进入特权模式
Router> enable
# 查看当前配置
Router# show running-config
# 进入配置模式
Router# configure
Router(config)

# 更改路由器名称
# 例如这里左边的路由器为 LR
Router(config)# hostname LR
LR(config)#

# 开始配置连接交换机和另一台路由器的网口

# 进入以太网口配置（连接交换机）
LR(config)# interface FastEthernet 0/0
# 分配 IP 地址
# 另一台路由器同理，但设置为 192.168.3.1
LR(config-if)# ip address 192.168.1.1 255.255.255.0
# 开启网口
LR(config-if)# no shutdown
# 退出此网口
LR(config-if)# exit

# 进入另一个网口（连接路由器）
LR(config)# interface FastEthernet 0/1
# 路由器不同网口要分配不同网段
# 另一台路由器网段必须相同，192.168.2.2
LR(config-if)# ip address 192.168.2.1 255.255.255.0
LR(config-if)# no shutdown
```

### 4. PC 终端配置

#### 分配 IP 地址

- 左边
    - IP 地址(IP Address): `192.168.1.100`
    - 默认网关(Default Gateway): `192.168.1.1`
    
    > 默认网关即默认路由器
    > 
    > 当 PC 机要给其他网络的计算机发送信息时，首先送到的同一个网络的路由器 IP 地址

- 右边
    - IP Address: `192.168.3.100`
    - Default Gateway: `192.168.3.1`

### 5. 测试连通

测试两台 PC 是否连通

在左边 PC 的 CMD 中：
```shell
PC> ping 192.168.3.100
# 失败！报错如下：
# Pinging 192.168.3.100 with 32 bytes of data:

# Reply from 192.168.1.1: Destination host unreachable.
# Request timed out.
# Reply from 192.168.1.1: Destination host unreachable.
# Reply from 192.168.1.1: Destination host unreachable.

# Ping statistics for 192.168.3.100:
#     Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

#### 查找失败原因

在左边的路由器中：

```shell
# 查看路由表
LR# show ip route 
# C    192.168.1.0/24 is directly connected, FastEthernet0/0
# C    192.168.2.0/24 is directly connected, FastEthernet0/1
```

LR 路由表中没有发现到达 192.168.3.* 的方法，因此数据包被丢弃了

## 配置路由表 - RIP 协议

相互交换路由器的路由表，用来计算路由信息，更新自己的路由表

```shell
# 启用 RIP 路由协议 并进入配置
LR(config)# router rip
LR(config-router)#
# 向邻居路由器发布自己的直连网络
# 第一版 RIP 路由协议不需要子网掩码
# 右边同理，但要给邻居发布的直连网络是 192.168.2.0 和 192.168.3.0
LR(config-router)# network 192.168.1.0
LR(config-router)# network 192.168.2.0
# 退出 RIP 配置
LR(config-router)# exit
LR(config)#

```

交换完成，等待路由信息计算完毕，重新查看路由表

```shell
# 查看路由表
LR# show ip route
# C    192.168.1.0/24 is directly connected, FastEthernet0/0
# C    192.168.2.0/24 is directly connected, FastEthernet0/1
# R    192.168.3.0/24 [120/1] via 192.168.2.2, 00:00:23, FastEthernet0/1
```

> 注意：[120/1] 中的120为 `管理代价`，表示路由协议的优先级
> 
> 数字越大，优先级越低

### 再次测试连通

在左边 PC 的 CMD 中：
```shell
PC> ping 192.168.3.100
```

---

> 单区域OSPF路由协议的配置

在 RIP 路由协议的基础上，开启 OSPF 路由协议

## 路由器配置 - OSPF 协议

配置 OSPF 路由

```shell
# 进入 OSPF 配置
# 由于一台路由器可以配置多个 OSPF 路由
# 因此需要指定 进程 ID，这里使用 1
LR(config)# router ospf <1-65535>
# 向邻居发布自己的直连网络
# OSPF 支持子网掩码，并需要划分区域
# !! 思科在这里采用反子网掩码，0 要写为 255，255 写为 1: 0.0.0.255
# !! 只有一个区域时必须是主区域，标记必须为 0.0.0.0，可以简写为 0
LR(config-router)# network 192.168.1.0 0.0.0.255 area <0-4294967295>
LR(config-router)# network 192.168.2.0 0.0.0.255 area <0-4294967295>

# 右边同理，注意进程号不互通，可以相同
```

查看路由表

```shell
RR# show ip route
# O    192.168.1.0/24 [110/2] via 192.168.2.1, 00:00:07, FastEthernet0/1
# C    192.168.2.0/24 is directly connected, FastEthernet0/1
# C    192.168.3.0/24 is directly connected, FastEthernet0/0
```

> 由于 OSPF 路由协议的优先级为高于 RIP, 因此覆盖了它的路由表信息
> 当 OSPF 路由故障时, RIP 路由协议的表项就会重新出现

### 测试连通

在左边 PC 的 CMD 中：
```shell
PC> ping 192.168.3.100
```

---

> 管理员手动添加静态路由

## 路由器配置 - 静态路由

在左边的路由器中：

```shell
# 添加静态路由：ip route <目的网络> <子网掩码> <下一跳路由器地址>
# 目的网络：网段 1 中的 PC0 只有网段 3 不知道如何到达：
# 下一跳路由器地址：必须是 PC0 直接能到的，这里为网段 2 的接口
LR(config)# ip route 192.168.3.0 255.255.255.0 192.168.2.2
LR(config)# exit
LR# show ip route
# Gateway of last resort is not set

# C    192.168.1.0/24 is directly connected, FastEthernet0/0
# C    192.168.2.0/24 is directly connected, FastEthernet0/1
# S    192.168.3.0/24 [1/0] via 192.168.2.2
```

> 静态路由优先级最高 [1/0]

### 测试连通

在左边 PC 的 CMD 中：
```shell
PC> ping 192.168.3.100
```

## 总结

配置顺序：`RIP` > `OSPF` > `静态路由`

