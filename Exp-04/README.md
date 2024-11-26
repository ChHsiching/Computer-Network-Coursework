# 访问控制列表 ACL

> 允许 - permit  
> 禁止 - deny

---

## 访问控制列表的基本原理

无论是 标准 / 扩展 访问控制列表，都可以指定多个条件 [permit / deny]

**这些条件的顺序非常重要**

路由器按由上到下的顺序依次匹配（以最上面的为准）

例如下面有一个访问控制列表：

```shell
permit ip1  # 在这里，路由器已经将数据包转发走，不会看到下面的禁止条件
deny ip2    # 不起作用
```

由于添加条件的顺序也是从上到下依次添加，因此添加新条件时，如果和前面的条件发生冲突，可以使用 `no` 进行删除

如果访问控制列表中没有对某 IP 规定允许 / 禁止，`默认丢弃`该 IP 的数据包。因为所有控制列表的最后都是 `deny any`

### 建立访问控制列表

1. 通过指定访问控制列表的 **名字** 以及其若干 **访问条件** 来建立访问控制列表

2. 将访问控制列表应用到路由器 **某个接口** 的 `In` 或 `Out` 上

> In / Out  
> 相对路由器，看数据流方向，流入为 In，流出为 Out

---

## 标准访问控制列表的配置

> 又称 `IP 访问列表`  
> 只针对`源 IP 地址`进行过滤，无法分析上层信息  
> 
> **标准 ACL 配置在离目的主机 <u>越近越好</u> 的路由器上**

### 1. 网络拓扑结构

#### 设备清单

- 路由器 * 2  `1841 Router`
- 交换机 * 2  `2950-24 Switch`
- 计算机 * 3  `PC-CT PC`
    - 分成两组：有两台 PC 连接在同一台交换机（左）

#### 设备连接

1. PC - Switch

    `直通线` 连接 `FastEthernet` 以太网口

2. Switch - Router

    `直通线` 连接 `FastEthernet` 以太网口

3. Router - Router

    `交叉线` 连接 `FastEthernet` 以太网口
    > 这里当然也可以使用 Serial 串口连接，但是我们不需要通过它进行 PPP 封装或者是 PAP、CHAP 认证，因此直接使用网口连接，简单快捷 

    由于 Cisco 思科 路由器的网口默认状态为 `shutdown`，需要手动打开

#### Router 路由器配置

> IP 地址的配置需满足的原则：
> - 不同网络的 IP 地址网络号不同
> - 同一网络的 IP 地址网络号相同

这里使用 `RIP 路由协议` 连通各个 PC 机

配置 RIP 路由协议的教程 >>> 1. [路由器初始化](../Exp-03/README.md#3-router-路由器配置) >>> 2. [PC机初始化](../Exp-03/README.md#4-pc-终端配置) >>> 3. [配置 RIP 路由协议](../Exp-03/README.md#配置路由表---rip-协议)

### 2. 配置标准访问控制列表

进入目的 PC 机（192.168.3.100）直连的路由器（右）

#### (1) 创建访问控制列表

```shell
# ip access-list <类型:standard/extended> <序号1-99/名字WORD>
# 标准 ACL 序号 1-99
# 扩展 ACL 序号 100-199
Router(config)# ip access-list standard 1
Router(config-std-nacl)#

# 禁止源 PC 机（左，192.168.1.100）对目的 PC 机的访问
# deny <源 IP 地址> <反子网掩码 wildcard>
Router(config-std-nacl)# deny 192.168.1.100 0.0.0.0
# 除了此主机外，其他全部放行
Router(config-std-nacl)# permit any
Router(config-std-nacl)# exit
Router(config)#
```

> 这里的反子网掩码中的 0 代表检查和匹配源 IP 地址的位数，例如：
>
> - 检查 192.168.1.100 的前 32 位: 0.0.0.0
> - 检查 192.168.1.100 的前 24 位: 0.0.0.255
> - 检查 192.168.1.100 的前 16 位: 0.0.255.255
> - 检查 192.168.1.100 的前  8 位: 0.255.255.255

#### (2) 将访问控制列表应用到路由器接口及其 Out 方向

选择离目的 PC 机最近的路由器以及最近的接口（数据流出的接口，Out）

```shell
# 选择离目的 PC 机最近的路由器以及最近的接口（数据流出的接口，Out）
Router(config)# interface FastEthernet 0/0
# access-group 用于指定数据包的访问控制
# 1 是访问控制列表的编号
# out 表示应用在接口的 out 上，另一个可选的值是 in
Router(config-if)# ip access-group 1 out
```

#### (3) 测试连通，验证访问控制列表

这里右侧路由器的 0/0(out方向接口) 拒绝了 192.168.1.100
但是其他的全部允许
因此测试 192.168.1.100 和 192.168.1.300 这两台主机是否能连接 192.168.3.100

- 在左边 192.168.1.100 主机的 CMD 中：

```shell
PC> ping 192.168.3.100

# Pinging 192.168.3.100 with 32 bytes of data:

# Reply from 192.168.2.2: Destination host unreachable.
# Reply from 192.168.2.2: Destination host unreachable.
# Reply from 192.168.2.2: Destination host unreachable.
# Reply from 192.168.2.2: Destination host unreachable.

# Ping statistics for 192.168.3.100:
#     Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

无法 ping 通，正确

- 在左边 192.168.1.200 主机的 CMD 中

```shell
PC> ping 192.168.3.100

# Pinging 192.168.3.100 with 32 bytes of data:

# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126

# Ping statistics for 192.168.3.100:
#     Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
# Approximate round trip times in milli-seconds:
#     Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

可以 ping 通，正确

---

## 扩展访问控制列表的配置

> 不仅可以针对数据包的`源 IP 地址`进行过滤  
> 还可以识别 `传输层协议类型` + `端口号`，并根据这些信息进行过滤
>
> **拓展 ACL 配置在离<u>源</u>主机 <u>越近越好</u> 的路由器上**  
> 因为拓展 ACL 判断更加准确，不会误丢弃数据包，因此越早越好

### 1. 网络拓扑结构

设备和连接与标准访问控制列表相同

唯一的区别是将右侧的 192.168.3.100 主机替换为服务器

#### 设备清单

- 路由器 * 2  `1841 Router`
- 交换机 * 2  `2950-24 Switch`
- 计算机 * 2  `PC-CT PC`
  - 两台主机连接在同一个路由器 
- 服务器 * 1  `Server-PT Server`
  - 服务器连接到另一台路由器

#### 设备连接


1. PC - Switch

    `直通线` 连接 `FastEthernet` 以太网口

2. Server -Switch

    `直通线` 连接 `FastEthernet` 以太网口

3. Switch - Router

    `直通线` 连接 `FastEthernet` 以太网口

4. Router - Router

    `交叉线` 连接 `FastEthernet` 以太网口
    > 这里当然也可以使用 Serial 串口连接，但是我们不需要通过它进行 PPP 封装或者是 PAP、CHAP 认证，因此直接使用网口连接，简单快捷 

    由于 Cisco 思科 路由器的网口默认状态为 `shutdown`，需要手动打开

#### Server 服务器配置

服务器配置为提供 HTTP 服务的 Web 服务器  

`配置` > `服务` > `HTTP`

HTTP 服务默认为启用状态，不需要修改

#### Router 路由器配置

> IP 地址的配置需满足的原则：
> - 不同网络的 IP 地址网络号不同
> - 同一网络的 IP 地址网络号相同

这里使用 `RIP 路由协议` 连通各个 PC 机

配置 RIP 路由协议的教程 >>> 1. [路由器初始化](../Exp-03/README.md#3-router-路由器配置) >>> 2. [PC机初始化](../Exp-03/README.md#4-pc-终端配置) >>> 3. [配置 RIP 路由协议](../Exp-03/README.md#配置路由表---rip-协议)

#### 测试连通

在 192.168.1.100 分别使用 CMD 和 浏览器 访问 192.168.3.100


### 2. 配置扩展访问控制列表

#### (1) 创建访问控制列表

这里我们将要配置的规则是：
- 禁止 192.168.1.100 对 192.168.3.100 的 Web 访问
- 允许其他所有 IP(包括 192.168.1.100) 对它的 ping 访问

在左边的路由器（直连源主机的路由器）上：

```shell
# ip access-list <类型:standard/extended> <序号1-99/名字WORD>
# 标准 ACL 序号 1-99
# 扩展 ACL 序号 100-199
Router(config)# ip access-list extended 100
Router(config-ext-nacl)#
# 添加条件：禁止 192.168.1.100 对 192.168.3.100 的 Web 访问
# 这里禁止 Web 访问，因此要禁止的是 HTTP 协议
# HTTP 协议基于传输层的 TCP 协议
# deny <协议> <源 IP 地址> <源 反子网掩码> <目的 IP 地址> <目的 反子网掩码> <禁用的协议的相应端口>
# 这里的反子网掩码和标准 ACL 作用相同
# 这里禁用的协议的相应端口有两个关键字，第一个是指定范围，第二个是值，eq是相等，除此之外还可以设置大于、小于、范围等等，可通过 ? 查看
Router(config-ext-nacl)# deny tcp 192.168.1.100 0.0.0.0 192.168.3.100 0.0.0.0 eq 80

# 允许其他所有访问，否则最后一行自动 deny any 
# 互联网所有访问都基于 IP 协议
# 允许任何源，任何目的的访问
Router(config-ext-nacl)# permit ip any any
# 这里也可以写得更加精准，只允许 ping：icmp 协议(Internet Control Message Protocol)
Router(config-ext-nacl)# permit icmp any any
```

#### (2) 将访问控制列表应用到路由器接口及其 In 方向

选择离源 PC 机最近的路由器以及最近的接口（数据流入的接口，In）

```shell
Router(config)# interface FastEthernet 0/0
Router(config-if)# ip access-group 100 in
```

#### (3) 测试连通，验证访问控制列表

这里左侧路由器的 0/0(in方向接口) 禁止 192.168.1.100 到 192.168.3.100 的 Web访问
但是 ping 是允许的
因此测试 192.168.1.100 和 192.168.1.300 这两台主机是否能连接 192.168.3.100

- 在左边 192.168.1.100 主机的 浏览器 中：访问 http://192.168.3.100

```
Request Timeout
```

无法连接，正确

- 在左边 192.168.1.100 主机的 CMD 中：

```shell
PC> ping 192.168.3.100

# Pinging 192.168.3.100 with 32 bytes of data:

# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=0ms TTL=126
# Reply from 192.168.3.100: bytes=32 time=1ms TTL=126

# Ping statistics for 192.168.3.100:
#     Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
# Approximate round trip times in milli-seconds:
#     Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

可以 ping 通，正确