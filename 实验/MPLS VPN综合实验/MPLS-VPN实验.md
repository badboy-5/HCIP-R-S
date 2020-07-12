---
姓名：坏坏
实验时间：2020年6月28日
整理时间：2020年6月28日
---

# MPLS VPN实验

如下拓扑：

![MPLS VPN实验拓扑](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\MPLS VPN实验拓扑.png)

## CE基础配置

```bash
#
 sysname CE1
#
interface GigabitEthernet0/0/0
 ip address 10.1.11.1 255.255.255.0 
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255 
```
```bash
#
 sysname CE2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255 
```
```bash
#
 sysname CE3
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.1 255.255.255.0 
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255
```
```bash
#
 sysname CE4
#
interface GigabitEthernet0/0/0
 ip address 10.1.24.1 255.255.255.0 
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255 
```

## PE基础配置

```bash
#
 sysname PE1
#
interface GigabitEthernet0/0/0
 ip address 200.1.100.1 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.11.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 10.1.12.2 255.255.255.0 
#
interface LoopBack0
 ip address 11.11.11.11 255.255.255.255
```
```bash
#
 sysname P
#
interface GigabitEthernet0/0/0
 ip address 200.1.100.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 200.1.200.3 255.255.255.0 
#
interface LoopBack0
 ip address 33.33.33.33 255.255.255.255 
```
```bash
#
 sysname PE2
#
interface GigabitEthernet0/0/0
 ip address 200.1.200.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 10.1.24.2 255.255.255.0 
#
interface LoopBack0
 ip address 22.22.22.22 255.255.255.255 
```

## ISP内配置IGP

```bash
# PE1
ospf 1 
 area 0.0.0.0 
  network 11.11.11.11 0.0.0.0 
  network 200.1.100.1 0.0.0.0 
```
```bash
# P
ospf 1 
 area 0.0.0.0 
  network 33.33.33.33 0.0.0.0 
  network 200.1.100.3 0.0.0.0 
  network 200.1.200.3 0.0.0.0 
```
```bash
# PE2
ospf 1 
 area 0.0.0.0 
  network 22.22.22.22 0.0.0.0 
  network 200.1.200.2 0.0.0.0 
```

## 配置MPLS

```bash
[PE1]mpls lsr-id 11.11.11.11
[PE1]mpls
[PE1-mpls]mpls ld 
[PE1-mpls-ldp]int g0/0/0
[PE1-GigabitEthernet0/0/0]mpls
[PE1-GigabitEthernet0/0/0]mpls ldp
```
```bash
[P]mpls lsr-id 33.33.33.33
[P]mpls
[P-mpls]mpls ld
[P-mpls-ldp]int g0/0/0
[P-GigabitEthernet0/0/0]mpls
[P-GigabitEthernet0/0/0]mpls ldp
[P-GigabitEthernet0/0/0]int g0/0/1
[P-GigabitEthernet0/0/1]mpls
[P-GigabitEthernet0/0/1]mpls ldp
```
```bash
[PE2]mpls lsr-id 22.22.22.22
[PE2]mpls
[PE2-mpls]mpls ld
[PE2-mpls-ldp]int g0/0/0
[PE2-GigabitEthernet0/0/0]mpls
[PE2-GigabitEthernet0/0/0]mpls ldp
```

> - `dis mpls lsp`查看MPLS邻居建立状态

## CE上配置IGP

```bash
# CE1
ospf 1 
 area 0.0.0.0 
  network 1.1.1.1 0.0.0.0 
  network 10.1.11.1 0.0.0.0
```
```bash
# CE2
ospf 1 
 area 0.0.0.0 
  network 2.2.2.2 0.0.0.0 
  network 10.1.12.1 0.0.0.0
```
```bash
# CE3
ospf 1 
 area 0.0.0.0 
  network 3.3.3.3 0.0.0.0 
  network 10.1.23.1 0.0.0.0 
```
```bash
# CE4
ospf 1 
 area 0.0.0.0 
  network 4.4.4.4 0.0.0.0 
  network 10.1.24.1 0.0.0.0 
```

## PE1配置VPN、与CE设备间的IGP

```bash
[PE1]ip vpn-instance vpn1  //创建vpn1
[PE1-vpn-instance-vpn1]route-distinguisher 1:1  //配置RD为1:1
[PE1-vpn-instance-vpn1-af-ipv4]vpn-target 1:3  //配置RT为1:3
[PE1-vpn-instance-vpn1-af-ipv4]dis th
[V200R003C00]
#
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:3 export-extcommunity
  vpn-target 1:3 import-extcommunity
```

> 配置RT时，不指定进出方向默认是都使用

```bash
[PE1]ospf 2 vpn-instance vpn1  //绑定vpn实例
[PE1-ospf-2]area 0
[PE1-ospf-2-area-0.0.0.0]net 10.1.11.2 0.0.0.0
[PE1-ospf-2-area-0.0.0.0]int g0/0/1
[PE1-GigabitEthernet0/0/1]ip binding vpn-instance vpn1  //接口绑定vpn实例
[PE1-GigabitEthernet0/0/1]ip ad 10.1.11.2 24  //重新配置IP地址
```

> 接口下绑定vpn实例后，会把原本接口下关于三层的配置全部清除，需要重新配置

```bash
# 为CE2配置vpn、IGP
[PE1]ip vpn-instance vpn2  //创建vpn2
[PE1-vpn-instance-vpn2]route-distinguisher 2:2  //配置RD为2:2
[PE1-vpn-instance-vpn2-af-ipv4]vpn-target 2:3  //配置RT为2:3
[PE1-vpn-instance-vpn2-af-ipv4]int g0/0/2
[PE1-GigabitEthernet0/0/2]ip binding vpn-instance vpn2  //接口i绑定vpn实例
[PE1-GigabitEthernet0/0/2]ip ad 10.1.12.2 24  //接口重新配置IP

[PE1]ospf 3 vpn-instance vpn2  //OSPF绑定vpn实例
[PE1-ospf-3]area 0
[PE1-ospf-3-area-0.0.0.0]net 10.1.12.2 0.0.0.0
```

## PE2配置VPN、与CE设备间的IGP

```bash
# 创建并配置VPN
[PE2]ip vpn-instance vpn1
[PE2-vpn-instance-vpn1]route-distinguisher 3:3
[PE2-vpn-instance-vpn1-af-ipv4]vpn-target 1:3
# 配置PE与CE间的IGP
[PE2]ospf 2 vpn-instance vpn1
[PE2-ospf-2]area 0
[PE2-ospf-2-area-0.0.0.0]net 10.1.23.2 0.0.0.0
# 接口绑定VPN实例
[PE2]int g0/0/1
[PE2-GigabitEthernet0/0/1]ip binding vpn-instance vpn1
[PE2-GigabitEthernet0/0/1]ip ad 10.1.23.2 24
```
```bash
[PE2]ip vpn-instance vpn2 
[PE2-vpn-instance-vpn2]route-distinguisher 4:4
[PE2-vpn-instance-vpn2-af-ipv4]vpn-target 2:3

[PE2]ospf 3 vpn-instance vpn2
[PE2-ospf-3]area 0    
[PE2-ospf-3-area-0.0.0.0]net 10.1.24.2 0.0.0.0

[PE2]int g0/0/2
[PE2-GigabitEthernet0/0/2]ip binding vpn-instance vpn2 
[PE2-GigabitEthernet0/0/2]ip ad 10.1.24.2 24
```

![全局路由表、vpn路由表](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\全局路由表、vpn路由表.png)

## 配置MP-BGP

```bash
# 配置普通BGP
[PE1]bgp 100
[PE1-bgp]peer 22.22.22.22 as-number 100
[PE1-bgp]peer 22.22.22.22 connect-interface LoopBack 0
# 激活传递VPNv4能力
[PE1-bgp]ipv4-family vpnv4
[PE1-bgp-af-vpnv4]peer 22.22.22.22 enable
```
```bash
[PE2]bgp 100
[PE2-bgp]peer 11.11.11.11 as-number 100
[PE2-bgp]peer 11.11.11.11 connect-interface LoopBack 0

[PE2-bgp]ipv4-family vpnv4
[PE2-bgp-af-vpnv4]peer 11.11.11.11 enable 
```

![VPNv4的邻居路由](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\VPNv4的邻居路由.png)

## 将CE与PE之间的IGP路由引入到MP-BGP路由表

```bash
[PE1]bgp 100
[PE1-bgp]ipv4-family vpn-instance vpn1
[PE1-bgp-vpn1]import-route ospf 2
[PE1-bgp-vpn1]q
[PE1-bgp]ipv4-family vpn-instance vpn2
[PE1-bgp-vpn2]import-route ospf 3
```

> MP-BGP的VPNv4路由表

![VPNv4路由表](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\VPNv4路由表.png)

```bash
# PE2将BGP路由引入OSPF
[PE2]ospf 2
[PE2-ospf-2]import-route bgp
[PE2-ospf-2]ospf 3
[PE2-ospf-3]import-route bgp
```

> - 在CE3（CE4）上查看全局路由表，看是否由1.1.1.1（2.2.2.2）的路由
> - 同样配置回路

```bash
# BGP引入OSPF
[PE2]bgp 100
[PE2-bgp]ipv4-family vpn-instance vpn1
[PE2-bgp-vpn1]import-route ospf 2
[PE2-bgp-vpn1]q  
[PE2-bgp]ipv4-family vpn-instance vpn2
[PE2-bgp-vpn2]import-route ospf 3
# OSPF引入BGP
[PE1]ospf 2
[PE1-ospf-2]import-route bgp
[PE1-ospf-2]ospf 3
[PE1-ospf-3]import-route bgp
```

> 在CE1`ping -a 1.1.1.1 3.3.3.3`测试连通性

## 抓包查看标签

![查看多层标签](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\查看多层标签.png)

## PE2的MP-BGP重新更新报文（`<PE2>refresh bgp all export`）

![MP-BGP更新报文](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验\MPLS-VPN实验.assets\MP-BGP更新报文.png)