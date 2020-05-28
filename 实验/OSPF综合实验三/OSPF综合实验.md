---
姓名：坏坏
实验时间：2020年5月26日
整理时间：2020年5月26日
---

实验拓扑[【下载】]()

# OSPF综合实验

如下拓扑图：


**实验需求**：

1. 按照图示配置 IP 地址，所有路由器配置环回口 IP 地址为 X.X.X.X/32 作为 Router-id，X 为设备编号（R1 除外）
2. 按照图示分区域配置 OSPF 与 RIP
3. R1 上配置环回口模拟业务网段；并在 R2 上进行单点双向路由引入，要求所有业务网段的路由聚合为一条后发布到 OSPF 区域
4. Area 3 不允许出现非直连的明细路由；并要求所有路由可达
5. PC1 访问业务网段路由需优先走以太网链路，并要求所在的直连网段不允许出现协议报文
6. OSPF 区域内路由器不允许存在 100.12.1.0/24 网段的路由
7. 为了保证协议安全，Area 0 配置区域验证，验证密钥 123456

## 基础配置

```bash
-----------------------------------------
# AR1基础配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 100.12.1.1 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.0.1 255.255.255.255 
#
interface LoopBack1
 ip address 192.168.1.1 255.255.255.255 
#
interface LoopBack2
 ip address 192.168.2.1 255.255.255.255 
#
interface LoopBack3
 ip address 192.168.3.1 255.255.255.255 
-----------------------------------------
# AR2基础配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 100.12.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 23.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 24.1.1.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255 
-----------------------------------------
# AR3基础配置
 sysname AR3
#
interface GigabitEthernet0/0/0
 ip address 23.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 35.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/2
#
interface NULL0
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255
-----------------------------------------
# AR4基础配置
 sysname AR4
#
interface Serial1/0/0
 link-protocol ppp
 ip address 45.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 24.1.1.4 255.255.255.0 
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255
-----------------------------------------
# AR5基础配置
 sysname AR5
#
interface Serial1/0/0
 link-protocol ppp
 ip address 45.1.1.5 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 35.1.1.5 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 56.1.1.5 255.255.255.0 
#
interface LoopBack0
 ip address 5.5.5.5 255.255.255.255
-----------------------------------------
# AR6基础配置
 sysname AR6
#
interface GigabitEthernet0/0/0
 ip address 56.1.1.6 255.255.255.0 
#
interface LoopBack0
 ip address 6.6.6.6 255.255.255.255
-----------------------------------------
```

## 配置OSPF、RIP

```bash

[AR1]rip
[AR1-rip-1]v 2
[AR1-rip-1]net 192.168.0.0          
[AR1-rip-1]net 192.168.1.0
[AR1-rip-1]net 192.168.2.0
[AR1-rip-1]net 192.168.3.0
[AR1-rip-1]net 100.0.0.0 

[AR2]rip
[AR2-rip-1]v 2
[AR2-rip-1]net 2.0.0.0
[AR2-rip-1]net 100.0.0.0
[AR2-rip-1]dis ip routing-table  //查看路由表

[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255

[AR3]ospf router-id 3.3.3.3
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]area 1
[AR3-ospf-1-area-0.0.0.1]net 35.1.1.0 0.0.0.255

[AR4]ospf router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0
[AR4-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]area 2
[AR4-ospf-1-area-0.0.0.2]net 45.1.1.0 0.0.0.255

[AR5]ospf router-id 5.5.5.5
[AR5-ospf-1]area 1
[AR5-ospf-1-area-0.0.0.1]net 5.5.5.5 0.0.0.0
[AR5-ospf-1-area-0.0.0.1]net 35.1.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.1]area 2
[AR5-ospf-1-area-0.0.0.2]net 45.1.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.2]net 172.16.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.2]area 3
[AR5-ospf-1-area-0.0.0.3]net 56.1.1.0 0.0.0.255

[AR6]ospf router-id 6.6.6.6
[AR6-ospf-1]area 3
[AR6-ospf-1-area-0.0.0.3]net 6.6.6.6 0.0.0.0
[AR6-ospf-1-area-0.0.0.3]net 56.1.1.0 0.0.0.255
```

## 单点引入

```bash
[AR2]ospf 1
[AR2-ospf-1]import-route rip 1
[AR2-ospf-1]rip
[AR2-rip-1]import-route ospf 1


```













