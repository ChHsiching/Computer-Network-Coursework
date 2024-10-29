# 常用命令及操作

---

## 1. Switch CLI - 交换机

### 常用命令
```shell
# 查看可用命令
Switch> ?
Switch> show ?

# 查看交换机配置
Switch> show running-config

# 查看 VLAN 划分
Switch> show vlan

# 特权模式
Switch> enable
Switch# disable
Switch>

# 配置模式
Switch# configure
Switch(config)# exit
Switch#

# 更改主机名称
Switch(config)# hostname S1
S1(config)#

# 划分 VLAN
Switch(config)# vlan <VID>
Switch(config-vlan)# exit
Switch(config)#

# 给 VLAN 分配端口
# 1. 进入待分配端口
Switch(config)# interface FastEthernet 0/<[1-24]>
Switch(config-if)#
# 2. 端口配置模式
# 端口加入 VLAN 的两种方式
# - access 此端口只可加入一个 VLAN
# - trunk  此端口可以加入多个 VLAN
Switch(config-if)# switchport access vlan <VID>
Switch(config-if)# exit
Switch(config)#
# 端口批量加入 VLAN
# 例如将 1 到 24 端口全部加入某个 VID 的 VLAN
Switch(config)# interface range FastEthernet 0/1-24
Switch(config-if-range)# switchport access vlan <VID>
Switch(config-if-range)# exit
Switch(config)#
```

## 2. 连线

- **不同**设备之间使用 `交叉线` 连接
- **相同**设备之间使用 `直通线` 连接

### VLAN 互访

> 交换机之间，交叉线两端的端口所属的 VLAN 可以不需要其他配置就能互相连通

- 使用一根交叉线，让交换机之间多个 VLAN 互访：将 `access` 改为 `trunk` 模式

```shell
# 进入端口配置
Switch(config)# interface FastEthernet 0/<交叉线连接的端口>
Switch(config-if)#

# 更改端口工作模式
Switch(config-if)# switchport mode trunk

# 指定端口属于哪些 VLAN
Switch(config-if)# switchport trunk allowed vlan <all/VID/...>
```



## 3. PC - 终端设备

1. 配置 IP 地址 - IP Configuration
- [ ] DHCP
- [x] Static

- IP Address (IP 地址): 192.168.1.<1-255>
> #### 如何保证设备在同一个C类网络中？
> - 保证前三段相同：192.168.1.*

- Subnet Mask(子网掩码): 255.255.255.0

