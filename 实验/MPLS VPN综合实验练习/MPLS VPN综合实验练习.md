---
姓名：坏坏
实验时间：2020年6月28日
整理时间：2020年6月28日
---

# MPLS VPN综合实验练习

如下拓扑图：

![MPLS VPN综合实验练习拓扑](F:\GitHub\HCIP R&S\实验\MPLS VPN综合实验练习\MPLS VPN综合实验练习.assets\MPLS VPN综合实验练习拓扑.png)

**实验需求**：

1. 如图，site A 和 site B 企业分别租用运营商的 MPLS 网络组建 BGP MPLS VPN，用来连接各自的分支机构
2. 按照图示配置 IP 地址，site A 公司和 site B 公司连接同一个 PE 设备的私网 IP 网段存在地址复用，使用多 VRF 技术来防止 IP 冲突
3. MPLS 网络中的设备配置 Loopback0 口用来建立 LSP 路径，使用 LDP 来建立 LSP 路径，各路由器使用 Loopback0 口作为 LSR-id
4. 公共网络中运行 OSPF 使公网连通，PE 设备宣告各自 Loopback0 口
5. 各自私网内部使用独立 OSPF 实例在 CE 和 PE 之间传递私网路由
6. 只允许在各自企业内部传递私网路由
7. 各自企业内部可以跨站点互相访问

## 基础配置

- CE设备配置

```bash
#
 sysname Site A-1
#
interface GigabitEthernet0/0/0
 ip address 10.1.11.1 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.1.1 255.255.255.0 
```
```bash
#
 sysname Site B-2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.2.1 255.255.255.0 
```
```bash
#
 sysname Site A-3
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.1 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.1.2 255.255.255.0 
```
```bash
#
 sysname Site B-4
#
interface GigabitEthernet0/0/0
 ip address 10.1.24.1 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.2.2 255.255.255.0 
```

- ISP设备IP配置

```bash
#
 sysname PE1
#
interface GigabitEthernet0/0/0
 ip address 200.1.13.1 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.11.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 10.1.12.2 255.255.255.0 
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
```
```bash
#
 sysname P
#
interface GigabitEthernet0/0/0
 ip address 200.1.13.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 200.1.23.3 255.255.255.0 
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255 
```
```bash
#
 sysname PE2
#
interface GigabitEthernet0/0/0
 ip address 200.1.23.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 10.1.24.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255 
```

## ISP网络配置OSPF

```bash
# PE1
ospf 1 
 area 0.0.0.0 
  network 1.1.1.1 0.0.0.0 
  network 200.1.13.1 0.0.0.0 
```
```bash
# P
ospf 1 
 area 0.0.0.0 
  network 3.3.3.3 0.0.0.0 
  network 200.1.13.3 0.0.0.0 
  network 200.1.23.3 0.0.0.0 
```
```bash
# PE2
ospf 1 
 area 0.0.0.0 
  network 2.2.2.2 0.0.0.0 
  network 200.1.23.2 0.0.0.0
```

> 查看OSPF邻居状态

## ISP网络怕配置MPLS

```bash
[PE1]mpls lsr-id 1.1.1.1
[PE1]mpls
[PE1-mpls]mpls ld
[PE1-mpls-ldp]int g0/0/0
[PE1-GigabitEthernet0/0/0]mpls
[PE1-GigabitEthernet0/0/0]mpls ldp
```
```bash
[P]mpls lsr-id 3.3.3.3
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
[PE2]mpls lsr-id 2.2.2.2
[PE2]mpls
[PE2-mpls]mpls ld 
[PE2-mpls-ldp]int g0/0/0
[PE2-GigabitEthernet0/0/0]mpls
[PE2-GigabitEthernet0/0/0]mpls ldp
```

## 在PE设备配置本地VPN，划分VRF，绑定相应接口

- 创建本地VPN A，配置相应的RD值、RT值

```bash
[PE1]ip vpn-instance A
[PE1-vpn-instance-A]route-distinguisher 100:100
[PE1-vpn-instance-A-af-ipv4]vpn-target 100:100 import-extcommunity
[PE1-vpn-instance-A-af-ipv4]vpn-target 200:200 export-extcommunity
```

- 将连接Site A-1站点的相应接口绑定到A的VPN实例，重新配置IP地址

```bash
[PE1]int g0/0/1
[PE1-GigabitEthernet0/0/1]ip binding vpn-instance A
[PE1-GigabitEthernet0/0/1]ip ad 10.1.11.2 24
```

> 同样的配置方法配置Site B-2

- 创建本地VPN B，配置相应的RD值、RT值

```bash
[PE1]ip vpn-instance B
[PE1-vpn-instance-B]route-distinguisher 300:300
[PE1-vpn-instance-B-af-ipv4]vpn-target 300:300 import-extcommunity
[PE1-vpn-instance-B-af-ipv4]vpn-target 400:400 export-extcommunity
```

- 将连接Site B-2站点的相应接口绑定到B的VPN实例，重新配置IP地址

```bash
[PE1]int g0/0/2
[PE1-GigabitEthernet0/0/2]ip binding vpn-instance B
[PE1-GigabitEthernet0/0/2]ip ad 10.1.12.2 24
```

> 同样方法配置PE2

- PE2配置

```bash
[PE2]ip vpn-instance A
[PE2-vpn-instance-A]route-distinguisher 100:100
[PE2-vpn-instance-A-af-ipv4]vpn-target 200:200 import-extcommunity
[PE2-vpn-instance-A-af-ipv4]vpn-target 100:100 export-extcommunity
[PE2-vpn-instance-A-af-ipv4]int g0/0/1
[PE2-GigabitEthernet0/0/1]ip binding vpn-instance A
[PE2-GigabitEthernet0/0/1]ip ad 10.1.23.2 24
```
```bash
[PE2]ip vpn-instance B
[PE2-vpn-instance-B]route-distinguisher 300:300
[PE2-vpn-instance-B-af-ipv4]vpn-target 400:400 import-extcommunity
[PE2-vpn-instance-B-af-ipv4]vpn-target 300:300 export-extcommunity
[PE2-vpn-instance-B-af-ipv4]int g0/0/2
[PE2-GigabitEthernet0/0/2]ip binding vpn-instance B
[PE2-GigabitEthernet0/0/2]ip ad 10.1.24.2 24
```

## 配置MP-BGP

- 配置BGP

```bash
[PE1]bgp 100
[PE1-bgp]peer 2.2.2.2 as-number 100
[PE1-bgp]peer 2.2.2.2 connect-interface LoopBack 0 
```

- 激活传递VPNv4的能力

```bash
[PE1-bgp]ipv4-family vpnv4
[PE1-bgp-af-vpnv4]peer 2.2.2.2 enable
```

> 同样方法配置PE2

- PE2配置MP-BGP

```bash
[PE2]bgp 100
[PE2-bgp]peer 1.1.1.1 as-number 100 
[PE2-bgp]peer 1.1.1.1 connect-interface LoopBack 0
[PE2-bgp]ipv4-family vpnv4
[PE2-bgp-af-vpnv4]peer 1.1.1.1 enable
```

## 配置私网内部的OSPF实例，将CE设备的路由传递给PE

- 在PE设备配置OSPF，绑定VPN实例

```bash
[PE1]ospf 2 vpn-instance A
[PE1-ospf-2]area 0
[PE1-ospf-2-area-0.0.0.0]net 10.1.11.2 0.0.0.0
```

- CE设备配置OSPF

```bash
[Site A-1]ospf
[Site A-1-ospf-1]area 0
[Site A-1-ospf-1-area-0.0.0.0]net 192.168.1.1 0.0.0.0
[Site A-1-ospf-1-area-0.0.0.0]net 10.1.11.1 0.0.0.0
```

> 同样配置Site B-2

- Site B-2与PE1之间配置

```bash
# PE1配置
[PE1]ospf 3 vpn-instance B
[PE1-ospf-3]area 0
[PE1-ospf-3-area-0.0.0.0]net 10.1.12.2 0.0.0.0
# Site B-2配置
[Site B-2]ospf
[Site B-2-ospf-1]area 0
[Site B-2-ospf-1-area-0.0.0.0]net 192.168.2.1 0.0.0.0
[Site B-2-ospf-1-area-0.0.0.0]net 10.1.12.1 0.0.0.0
```

> 同样方法配置PE2与CE之间的OSPF

- PE2与CE之间配置

```bash
[PE2]ospf 2 vpn-instance A
[PE2-ospf-2]area 0
[PE2-ospf-2-area-0.0.0.0]net 10.1.23.2 0.0.0.0

[Site A-3]ospf
[Site A-3-ospf-1]area 0
[Site A-3-ospf-1-area-0.0.0.0]net 192.168.1.2 0.0.0.0
[Site A-3-ospf-1-area-0.0.0.0]net 10.1.23.1 0.0.0.0
--------------------------------------------------------
[PE2]ospf 3 vpn-instance B
[PE2-ospf-3]area 0
[PE2-ospf-3-area-0.0.0.0]net 10.1.24.2 0.0.0.0

[Site B-4]ospf
[Site B-4-ospf-1]area 0
[Site B-4-ospf-1-area-0.0.0.0]net 192.168.2.2 0.0.0.0
[Site B-4-ospf-1-area-0.0.0.0]net 10.1.24.1 0.0.0.0
```

> - `[PE1]dis ospf peer bri`查看OSPF路由表
> - `[PE1]dis ip routing-table vpn-instance A`查看VPN A的路由表

## 配置单点双向引入

- PE1配置

```bash
# BGP中引入OSPF
[PE1]bgp 100 
[PE1-bgp]ipv4-family vpn-instance A
[PE1-bgp-A]import-route ospf 2
[PE1-bgp-A]q
[PE1-bgp]ipv4-family vpn-instance B
[PE1-bgp-B]import-route ospf 3
# OSPF中引入BGP
[PE1-bgp-B]ospf 2
[PE1-ospf-2]import-route bgp
[PE1-ospf-2]ospf 3
[PE1-ospf-3]import-route bgp
```

- PE2配置

```bash
# BGP中引入OSPF
[PE2]bgp 100
[PE2-bgp]ipv4-family vpn-instance A
[PE2-bgp-A]import-route ospf 2
[PE2-bgp-A]q
[PE2-bgp]ipv4-family vpn-instance B
[PE2-bgp-B]import-route ospf 3
# OSPF中引入BGP
[PE2-bgp-B]ospf 2
[PE2-ospf-2]import-route bgp
[PE2-ospf-2]ospf 3
[PE2-ospf-3]import-route bgp
```

## 测试

- 在Site A-1上`ping -a 192.168.1.1 192.168.1.2`测试与Site A-3、Site B-2、Site B-4的连通性
- 测试结果：Site A-x能互通，Site B-x能互通