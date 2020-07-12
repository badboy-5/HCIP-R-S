---
姓名：坏坏
学习时间：2020年6月24日
整理时间：2020年6月27日
---

# MPLS

- 互联网流量快速增长，由于**硬件技术**限制，路由器采用**最长匹配算法**逐跳转发数据包，成为网络数据转发的瓶颈
- MPLS—多协议标签交换，采用**短而定长的标签**进行数据转发
- 属于**“2.5”**层协议，介于二层头部和三层头部之间

## MPLS基本网络架构

![MPLS基本网络架构](F:\GitHub\HCIP R&S\MPLS.assets\MPLS基本网络架构.png)

- MPLS网络中路由器的角色：
	* LER（Label Edge Router）：用于标签的压入或弹出，如RTB、RTD
	* LSR（Label Switched Router）：用于标签交换，如RTC
- LSP（Label Switched Path）：标签交换路径，即到达同一目的地址的报文在MPLS网络中经过的路径
- FEC（Forwarding Equivalent Class）：一般指具有相同转发处理方式的报文，在MPLS网络中，到达同一目的地址的所有报文就是一个FEC，这个目的地址，也是一个FEC
	* FEC的划分方式灵活，可以是源地址、目的地址、源端口、目的端口、协议类型或VPN等划分为任意组合

## MPLS体系结构

- > 控制平面：路由
- 转发平面：转发流量

![MPLS体系结构](F:\GitHub\HCIP R&S\MPLS.assets\MPLS体系结构.png)

> - 没有RIB（路由表），不会进行LDP（标签分发）
> - LDP相当于路由协议，LIB相当于路由表
> - 当控制平面有了路由表，可以下发到转发平面的RIB

## MPLS数据报文结构

- MPLS标签封装在链路层和网络层之间，支持任意的链路层协议
- Label（标签）：20Bit，短而定长的、只有本地意义的标识用于标识去往同一目的地址的报文分组
- Exp：3Bit，Qos，用于拓展
- S：1Bit，位位1时表明为最底层（靠近IP层）
- TTL：8Bit，和IP报文中的TTL（Time To Live）相同

## LSP建立

- 标签分配的方式：下游为上游分配标签
- 靠近路由起源的路由器为`Ingress`（入栈点）—>`Transit`（转发栈点）—>`Egress`（出栈点）

## 静态LSP

- 用户通过手工方式各个转发等价类分配标签建立转发隧道
- 静态LSP特点：
	* 不使用标签发布协议，不需要交互控制报文，资源消耗较小
	* 通过静态方式建立的LSP不能根据网络拓扑变化动态调整，需要管理员干预
- 静态LSP适用于拓扑结构简单并且稳定的网络

**实验**：如下拓扑

![静态LSP实验拓扑](F:\GitHub\HCIP R&S\MPLS.assets\静态LSP实验拓扑.png)

- 基础配置

```bash
#
 sysname R1
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
 ospf enable 1 area 0.0.0.0
#
ospf 1 
 area 0.0.0.0 
```

```bash
#
 sysname R2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
ospf 1 
 area 0.0.0.0 
```

```bash
#
 sysname R3
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.3 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
interface GigabitEthernet0/0/1
 ip address 10.1.34.3 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
ospf 1 
 area 0.0.0.0 
```

```bash
#
 sysname R4
#
interface GigabitEthernet0/0/0
 ip address 10.1.34.4 255.255.255.0 
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255 
#
ospf 1 
 area 0.0.0.0 
  network 4.4.4.4 0.0.0.0 
  network 10.1.34.4 0.0.0.0 
```

- 配置MPLS

```bash
[R1]mpls lsr-id 1.1.1.1
[R1]mpls  //开启MPLS协议
--------------------------------
[R2]mpls lsr-i
[R2]mpls lsr-id 2.2.2.2
--------------------------------
[R3]mpls ls
[R3]mpls lsr-id 3.3.3.3
--------------------------------
[R4]mpls lsr
[R4]mpls lsr-id 4.4.4.4
```

> - 静态LSP中lsr-id可以随便指定，即使本地没有这个地址，但是必须配
> - 需要配置两条LSP，一去一回

- 配置R1到R4的LSP

```bash
# R1配置静态LSP，角色为ingress，名字为1-4，目的为4.4.4.4，下一跳是10.1.12.2，出标签为102
[R1]static-lsp ingress 1-4 destination 4.4.4.4 32 nexthop 10.1.12.2 out-label 102
```
```bash
# R2配置静态LSP，角色为transit，名字为1-4，进接口为g0/0/0，进标签为102，下一跳为10.1.23.3，出标签为203
[R2]static-lsp transit 1-4 incoming-interface g0/0/0 in-label 102 nexthop 10.1.23.3 out-label 203
```
```bash
# R3配置静态LSP，角色为transit，名字为1-4，进接口为g0/0/0，进标签为203，下一跳为10.1.34.4，出标签为304
[R3]static-lsp transit 1-4 incoming-interface g0/0/0 in-label 203 nexthop 10.1.34.4 out-label 304
```
```bash
# R4配置静态LSP，角色为egress，名字为1-4，进接口为g0/0/0，进标签为304
[R4]static-lsp egress 1-4 incoming-interface g0/0/0 in-label 304
```

- 配置R4到R1的LSP

```bash
[R4]static-lsp ingress 4-1 destination 1.1.1.1 32 nexthop 10.1.34.3 out-label 403
[R3]static-lsp transit 4-1 incoming-interface g0/0/1 in-label 403 nexthop 10.1.23.2 out-label 302
[R2]static-lsp transit 4-1 incoming-interface g0/0/1 in-label 302 nexthop 10.1.12.1 out-label 201
[R1]static-lsp egress 4-1 incoming-interface g0/0/0 in-label 201 
```

> 配好LSP后，还需要在接口下开启MPLS才可以互通

- 接口下开启MPLS

```bash
[R1]int g0/0/0
[R1-GigabitEthernet0/0/0]mpls
------------------------------------------
[R2]int g0/0/0
[R2-GigabitEthernet0/0/0]mpls
[R2-GigabitEthernet0/0/0]int g0/0/1
[R2-GigabitEthernet0/0/1]mpls
------------------------------------------
[R3]int g0/0/0
[R3-GigabitEthernet0/0/0]mpls
[R3-GigabitEthernet0/0/0]int g0/0/1
[R3-GigabitEthernet0/0/1]mpls
------------------------------------------
[R4]int g0/0/0
[R4-GigabitEthernet0/0/0]mpls
```

- 测试

```bash
[R1]ping lsp -a 1.1.1.1 ip 4.4.4.4 32
```

![静态LSP路由跟踪](F:\GitHub\HCIP R&S\MPLS.assets\静态LSP路由跟踪.png)

![TunnelID信息](F:\GitHub\HCIP R&S\MPLS.assets\TunnelID信息.png)

> - 决定是IP转发还是走隧道转发看TunnelID
> 	* 当TunnelID为0x0时，走IP转发
> 	* 当TunnelID不为0x0时，走隧道转发，具体走那个隧道看TunnelID对应的隧道

## 动态LSP

- 通过标签发布协议动态建立转发隧道
- 通过LDP协议实现对FEC的分类、标签的分配以及LSP的建立和维护
- 动态LSP的特点：
	* 组网配置简单，易于管理和维护
	* 支持基于路由动态建立LSP，网络拓扑发生变化时，能及时反映网络状况
- LDP协议属于应用层协议，即基于TCP，也基于UDP，端口号固定为646

**LDP邻居发现**

![LDP邻居发现](F:\GitHub\HCIP R&S\MPLS.assets\LDP邻居发现.png)

- 为了能使开启了LDP的设备快速发现邻居，Hello报文基于UDP
- 为了保证邻居的有效性和可靠性，Hello报文周期（5s）发送，使用组播224.0.0.2作为目的IP
- Hello报文中携带Transport Address（传输地址）字段，与设备配置的LSR ID一致，表明与对端建立邻居关系时所使用的IP地址
- Transport Address字段的IP地址是物理接口地址则直接建立邻居关系，环回口地址必须保证路由可达

> 由传输IP地址大的一方发起TCP连接

![LDP邻居发现-2](F:\GitHub\HCIP R&S\MPLS.assets\LDP邻居发现-2.png)

- **Discovery**（发现阶段）
	* 基于UDP，发送Hello报文，发现邻居
- **TCP**（建立TCP连接阶段）
	- 基于TCP，建立TCP连接
- **Session**（会话阶段）
	* Initialization：向对端发送报文
	* Initialization+Keep alive：对端收到后，检查报文内容，没有问题会回复Initialization报文，并发送Keep alive报文确认
	* Keep alive：收到对端的Initialization报文后，同样检查报文内容，没有问题会恢复Keep alive
- **Advertisement**（通告阶段）
	* Address：LDP是运行在接口下，所以会发布运行了LDP的接口的IP
	* Label Mapping（标签映射关系）：包含标签所对应的FEC
- **Notification**阶段
	* 在检查Initialization报文时，如果发现报文中的信息不一致，则会回复Notification报文

## 标签的发布方式

![标签的发布方式](F:\GitHub\HCIP R&S\MPLS.assets\标签的发布方式.png)

**下游自主**

- DU（Downstream Unsolicited，下游自主方式）
- 对于一个到达同一目的地址报文的分组，LSR无需从上游获得标签请求消息，即可进行标签分配与分发

**下游按需**

- DoD（Downstream on Demand，下游按需方式）
- 对于一个到达同一目的地址报文的分组，LSR获得标签请求消息之后才进行标签分配与分发

> - 华为设备默认采用DU的方式发布标签
> - DU无需等待上游的请求消息，直接向邻居分配标签
> - 网络拓扑发生变化时，DU方式收敛时间更短

## 标签的分配控制方式

![标签的分配控制方式](F:\GitHub\HCIP R&S\MPLS.assets\标签的分配控制方式.png)

**独立标签分配控制方式**

- Independent，只要本地有关于此IP的路由，就会向上游分配此IP分组的标签，不用等待下游的标签


**有序标签分配控制方式**

- Ordered，当LSR具有此IP的下一跳标签或者该LSR就是此IP分组的出节点时，下游才会给上游发送此IP分组的标签

## 标签的保持方式

**自由标签保持方式**

- Liberal，对于从邻居LSR收到的标签映射，无论邻居LSR是不是自己的下一跳都保留

**保守标签保持方式**

- Conservative，对于从邻居LSR收到的标签映射，只有当邻居LSR是自己的下一跳才保留

> - 华为设备默认使用自由标签保持方式保存标签
> - 华为默认使用的搭配：发布方式使用DU（下游自主方式），控制方式使用Ordered（有序标签分配控制方式），保持方式使用Liberal（自由标签保持方式）

## LDP建立LSP过程

- IGP协议负责实现MPLS网络内路由可达，为FEC的分组提供路由
- LDP协议负责实现对FEC的分类、标签的分配以及LSP的建立和维护

## Penultimate Hop Popping

- PHP，倒数第二跳弹出
	* 倒数第二跳路由器收到带有标签的报文后，查找LFIB表，发现分配的出标签为**隐式空标签**3，于是执行弹出标签的动作，并将IP数据包转发给下游路由器

> 标签0-15不能使用

**实验**：如下拓扑图

![动态LSP拓扑](F:\GitHub\HCIP R&S\MPLS.assets\动态LSP拓扑.png)

- 基础配置

```bash
#
 sysname R1
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255 
-----------------------------------------
#
 sysname R2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255 
-----------------------------------------
#
 sysname R3
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.3 255.255.255.0
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255 
```

- OSPF配置

```bash
# R1
ospf 1 
 area 0.0.0.0 
  network 1.1.1.1 0.0.0.0 
  network 10.1.12.1 0.0.0.0 
-----------------------------------------
# R2
ospf 1 
 area 0.0.0.0 
  network 2.2.2.2 0.0.0.0 
  network 10.1.12.2 0.0.0.0 
  network 10.1.23.2 0.0.0.0
-----------------------------------------
# R3
ospf 1 
 area 0.0.0.0 
  network 3.3.3.3 0.0.0.0 
  network 10.1.23.3 0.0.0.0
```

- 配置MPLS

```bash
[R1]mpls lsr-id 1.1.1.1
[R1]mpls
Info: Mpls starting, please wait... OK!
[R1-mpls]mpls ld
[R1-mpls-ldp]int g0/0/0
[R1-GigabitEthernet0/0/0]mpls
[R1-GigabitEthernet0/0/0]mpls ldp
```

```bash
[R2]mpls lsr-id 2.2.2.2
[R2]mpls
Info: Mpls starting, please wait... OK!
[R2-mpls]mpls ld
[R2-mpls-ldp]int g0/0/0
[R2-GigabitEthernet0/0/0]mpls
[R2-GigabitEthernet0/0/0]mpls ldp
[R2-GigabitEthernet0/0/0]int g0/0/1
[R2-GigabitEthernet0/0/1]mpls
[R2-GigabitEthernet0/0/1]mpls ldp
```

```bash
[R3]mpls lsr-id 3.3.3.3
[R3]mpls
Info: Mpls starting, please wait... OK!
[R3-mpls]mpls ld
[R3-mpls-ldp]int g0/0/0
[R3-GigabitEthernet0/0/0]mpls
[R3-GigabitEthernet0/0/0]mpls ldp
```

- 测试

![动态LSP](F:\GitHub\HCIP R&S\MPLS.assets\动态LSP.png)