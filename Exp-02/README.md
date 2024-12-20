# Exp2 - PPP 协议

---

> 路由器串口的PPP封装

## Router - 路由器

### Serial - 串口

1. 为路由器添加串口

> 路由器初始状态下只有两个以太网接口 FastEthernet

- 首先按下开关将路由器关机
- 然后拖动 `WIC-2T` 模块到路由器的 `SLOT` 空槽中
- 再次按下开关，开机

2. 通过串口连接路由器

- `DCE`：数据电路终端设备
- `DTE`：数据终端设备
> - DCE设备在模拟软件中有时钟标识，负责速率的设置
> - 连接它们的串口线模拟运营商的传输网络

- DCE 串口线
  - 起始端连接 DCE 设备
  - 结束端连接 DTE 设备

- DTE 串口线
  - 起始端连接 DTE 设备
  - 结束端连接 DCE 设备

### CLI - PPP 封装配置

#### 常用命令

```shell
# ontinue with configuration dialog? [yes/no]: 
# dialog 方式功能受限，选择 no

# 进入特权模式
Router> enable
Router#

# 查看路由器配置
# 注意接口配置 interface 初始状态全部是 shutdown，配置完毕需要打开接口
Router# show running-config

# 进入配置模式
Router# configure
Router(config)#

# 更改路由器名称
Router(config)# hostname R1
R1(config)#

# 进入串口配置
R1(config)# interface serial <串口号，如 0/1/0>
R1(config-if)#

# 配置串口 IP 地址
# IP 地址：192.168.1.<1-255>
# 子网掩码：255.255.255.0
R1(config-if)# ip address <IP地址> <子网掩码>

# 封装接口
R1(config-if)# encapsulation ppp

# *[Option] DCE 设备需要进行速率设置
# 通过 `clock rate ?` 查看可用速率，最常用的是 64000 bits per second
R1(config-if)# clock rate <300-4000000>

# 启动接口
# 路由器接口默认状态为 `shutdown`
R1(config-if)# no shutdown
# DCE 设备回显：
%LINK-5-CHANGED: Interface Serial0/1/0, changed state to down
# DTE 设备回显：
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/1/0, changed state to up
```

#### 测试连通

```shell
R1# ping 192.168.1.2

# 叹号(!)代表数据包收到，点(.)代表数据包丢失
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/8 ms
```

---

> 路由器串口PPP-PAP认证

#### 常用命令

```shell
# 进入特权模式
R1> enable
# 进入配置模式
R1# configure
R1(config)#

# 开启 3A 认证功能
# 3A: 认证 authentication, 授权 authorization, 计费 accounting
R1(config)# aaa new-model

# 建立用于认证的数据库（存放合法的用户名和口令字）
# 数据库既可以使用 enable 中的用户名和密码，也可以使用本地 local 数据库
# 这里使用 local
R1(config)# aaa authentication ppp <数据库名> <enable/local>
# 设置用户名和密码
# 这里使用 r2，p2
R1(config)# username <用户名> password <密码>

# 进入串口
R1(config)# interface serial <串口号，如 0/1/0>
R1(config-if)#

# 在 PPP 配置成功的基础上开启 PAP 认证
R1(config-if)# ppp authentication pap
# 配置 PAP 认证时发送到对方的用户名和口令字
# 这里使用 r1，p1
R1(config-if)# ppp pap sent-username <用户名> password <密码>
```

#### 测试连通

```shell
# 在特权模式下
R1# ping 192.168.1.2
```

---

> 路由器串口PPP-CHAP认证

#### 常用命令
```shell
# 进入特权模式
R1> enable
# 进入配置模式
R1# configure
R1(config)#

# 开启 3A 认证功能
R1(config)# aaa new-model

# 建立用于认证的数据库（存放合法的用户名和口令字）
# 这里使用 local
R1(config)# aaa authentication ppp <数据库名> <enable/local>
# 设置用户名和密码
# !! CHAP 认证时，用户名必须是对方的路由器名，口令字必须相同
# 这里使用 R2，p
R1(config)# username <用户名> password <密码>

# 进入串口
R1(config)# interface serial <串口号，如 0/1/0>
R1(config-if)#

# 在 PPP 配置成功的基础上开启 CHAP 认证
R1(config-if)# ppp authentication chap
```
> CHAP 认证时，用户名必须是对方的路由器名，口令字必须相同

#### 测试连通

```shell
# 在特权模式下
R1# ping 192.168.1.2
```
