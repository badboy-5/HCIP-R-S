---
姓名：坏坏
学习时间：2020年6月27日
整理时间：2020年6月27日
---

# MPLS VPN

- 专线的特点：
	* 线路专有，安全性高，不同用户之间物理隔离
	* 价格昂贵
	* 使用不充分，带宽浪费严重
- VPN的特点
	* 使用共享的公共网络环境实现各私网连接
	* 不同的私有网络之间互相不可见
- 设备名命：
	* ISP运营商网络中，边界的设备命名为PE，中间设备命名为P
	* 客户端设备命名为CE

## VPN模型—Overlay VPN

- 在CE与CE之间建立隧道，客户路由协议总是在客户设备之间交换，而运营商对客户网络结构一无所知
	* 优点：不同的客户地址空间可以重叠，保密性、安全性很好
	* 缺点：本质是一种“静态”VPN，无法反映网络的实时变化。配置与维护复杂，不易管理
- 在PE上为每一个VPN用户建立相应隧道，路由信息在PE与PE之间传递，公网中的P设备不知道私网的路由信息
	* 优点：客户把VPN的创建域维护完全交给运营商，保密性、安全性比较好
	* 缺点：不同的VPN用户不能共享相同的地址空间
- 典型的协议：
	* 二层：帧中继（淘汰）
	* 三层：GRE和IPSec
	* 应用层：SSL VPN

## VPN模型—Peer-To-Peer VPN

- Peer-To-Peer VPN是CE与PE设备之间交换私网信息，由PE设备将私网信息在运营商网络中传播，实现VPN部署及路由发布的动态性
- 解决了Overlay VPN的“静态”性质不适合大规模应用和部署的问题
- 所有VPN用户的CE设备连接到同一台PE上的缺点：
	* 防止一台PE设备上的不同CE之间互通，PE上需要配置大量的ACL，增加了管理PE设备的负担
	* VPN客户之间如果出现私网地址重叠的问题，PE设备无法识别
- 专用PE接入方式：运营商为每一个VPN单独准备一台PE设备，PE与CE之间可以运行任意的路由协议，与其他VPN无关
	* 优点：无需配置任何ACL，配置复杂度、管理难度有所降低
	* 缺点：每新增一个VPN站点，都需要新增一台专用的PE设备，价格昂贵，没有解决VPN客户之间地址空间重叠的问题

## MPLS VPN解决在网络传递过程中区分冲突路由

- RD（Route Distinguisher），在VPN路由发布到全局路由表之前，使用一个全局唯一的标识（RD）和路由绑定，以区分冲突与的私网路由
- 增加了RD的IPv4地址成为VPN-IPv4地址
- 运营商采用BGP协议作为承载VPN路由的协议，并将BGP协议进行拓展，成为MP-BGP

## Hub-Spoke场景中VPN路由引入问题

- RT（Route Target）属性用于将路由正确引入VPN，有两类VPN Target属性
	* Export Target：本端的路由在导出VRF，转变为VPNv4的路由时，标记的属性
	* Import Target：对端收到路由时，检查Export Target属性，当属性与VPN实例的Import Target匹配时，PE就把路由加入到该VPN实例中

## MPLS标签嵌套

- Outer MPLS Label在MPLS VPN中被称为公网标签，用于MPLS网络中转发数据。公网标签在到达PE设备时，已经在倒数第二层剥离掉
- Inner MPLS Label在MPLS VPN中被成为私网标签，用于将数据正确的发送到相应的VPN中。PE依靠Inner Label区分数据包属于哪个VPN

## MPLS VPN工作过程

**MPLS VPN路由传递过程**

- CE与PE之间路由交换（静态路由协议、动态路由协议）
- VRF路由注入MP-BGP的过程
	* VRF中的IPv4路由被添加上RD、RT与标签等信息成为VPNv4的路由放入到MP-BGP的路由表中，并通过MP-BGP协议在PE设备之间交换路由信息
- 公网标签的分配过程
	* MPLS协议在运营商网络分配公网标签，建立标签隧道，实现私网数据在公网上转发
- MP-BGP路由注入VRF的过程
	* PE之间运行MP-BGP协议为VPN路由分配私网标签，PE设备根据私网标签将数据正确转发给相应的VPN

**MPLS VPN数据的转发过程**

- CE设备到PE设备的数据转发
	* PE2收到数据包后，查找本地相应的VPN路由表，发现数据包需要进行标签转发，分配私网标签。到达目标地址的下一跳为PE1
	* PE2通过查找LFIB表，发现要通过隧道转发，PE2将数据进行MPLS封装
- 公网设备上的数据装发
	* PE2收到VPN用户的数据包后，封装上MPLS标签，将私网的数据通过MPLS建立的标签隧道进行转发
	* 公网标签剥离后，将数据包发送给PE1，PE1收到只有内层私网标签的数据包
- PE到CE设备的数据转发
	* PE1收到剥离公网标签的数据包后，根据根据私网标签查找转发数据包的下一跳，将数据包正确发送给相应VPN用户

# MPLS VPN实验

如下拓扑：

![MPLS VPN实验拓扑](F:\GitHub\HCIP R&S\MPLS-VPN.assets\MPLS VPN实验拓扑.png)

- CE基础配置

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

- PE基础配置

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

- ISP内配置IGP

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

- 配置MPLS

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

- CE上配置IGP

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

- PE1配置VPN、与CE设备间的IGP

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

- PE2配置VPN、与CE设备间的IGP

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

![全局路由表、vpn路由表](F:\GitHub\HCIP R&S\MPLS-VPN.assets\全局路由表、vpn路由表.png)

- 配置MP-BGP

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

![VPNv4的邻居路由](F:\GitHub\HCIP R&S\MPLS-VPN.assets\VPNv4的邻居路由.png)

- 将CE与PE之间的IGP路由引入到MP-BGP路由表

```bash
[PE1]bgp 100
[PE1-bgp]ipv4-family vpn-instance vpn1
[PE1-bgp-vpn1]import-route ospf 2
[PE1-bgp-vpn1]q
[PE1-bgp]ipv4-family vpn-instance vpn2
[PE1-bgp-vpn2]import-route ospf 3
```

> MP-BGP的VPNv4路由表

![VPNv4路由表](F:\GitHub\HCIP R&S\MPLS-VPN.assets\VPNv4路由表.png)

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

- 抓包查看标签

![查看多层标签](F:\GitHub\HCIP R&S\MPLS-VPN.assets\查看多层标签.png)

- PE2的MP-BGP重新更新报文（`<PE2>refresh bgp all export`）

![MP-BGP更新报文](F:\GitHub\HCIP R&S\MPLS-VPN.assets\MP-BGP更新报文.png)

