姓名：坏坏
学习时间：2020年5月11日
整理时间：2020年5月13日

# OSPF基原理

- OSPF(Open Shortest Path First)开放式最短路径优先，基于SPF算法的链路状态路由协议
- OSPF优点：
	* 路由信息传递与路由计算分离
	* 基于SPF算法（最短路径树）
	* 以“累计链路开销”作为选路参考值

**RIP在大型网络中部署面临的问题**

| RIP特性 | 问题 |
|:---:|:---|
| 逐跳收敛 | 收敛慢，故障恢复时间长 |
| 传闻路由更新机制 | 缺少对全局网络拓扑的了解 |
| 最多有效跳数为15 | 环形组网中，使远端路由不可达 |
| 以“跳数为度量” | 存在选择次优路径的风险 |

**OSPF相比于RIP的优化**

| RIP问题 | 优化 |
|:---:|:---|
| 收敛慢，故障恢复时间长 | 收到更新->计算路由->发送更新 ，更改为，收到更新->发送更新->计算路由 |
| 缺少对全局网络拓扑的了解 | 路由器基于拓扑信息，独立计算路由 |
| 最多有效跳数为15 | 不限定跳数 |
| 存在选择次优路径的风险 | 将链路带宽作为选路参考值 |

> - OSPF中收到路由信息放入DB（数据库）中，将数据库转发给邻居，同时放入协议RIB（路由表），计算路由，再加入RIB（全局路由表）
> - RIP收到路由信息，放入DB中（RIP的DB和协议RIB相同），计算好路由后，放入RIB（全局路由表），再转发给邻居

**OSPF工作过程**
- 邻居建立
- 同步链路状态数据库
- 计算最优路径

## 邻居建立过程

- Router-ID用于在自治系统中，唯一标识一台运行OSPF的路由器
	* 如果没有指定Router-ID，会优先从本地的环回口中选择最大的IP地址
	* 如果没有本地环回口地址，会从物理接口中选择最大的IP地址作为Router-ID

**发现并建立邻居**-发送Hello报文

- Hello报文的作用：
	* 邻居发现：自动发现邻居路由器
	* 邻居建立：完成Hello报文中的参数协商，建立令居关系
	* 邻居保持：通过保活机制检测邻居运行状态

![OSPF发现并建立邻居](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5COSPF%E5%8F%91%E7%8E%B0%E5%B9%B6%E5%BB%BA%E7%AB%8B%E9%82%BB%E5%B1%85.png)

> - 双方互相发送Hello报文，收到报文后，状态从Down进入Init，然后回复带有邻居Router-ID的Hello报文，邻居收到后，状态进入2-Way，邻居关系建立
> - **维护邻居关系**：
> 	* 广播和P2P每间隔10秒互相发送Hello报文，40秒（间隔时间的4倍）未收到Hello报文，就认为邻居出现故障
> 	* NBMA和P2MP每间隔30秒互相发送Hello报文，120秒未收到Hello报文就认为邻居出现故障
> - Hello报文通过组播（224.0.0.5）发送给其他路由器

**实验1**：如下拓扑

![试验1拓扑](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5C%E8%AF%95%E9%AA%8C1%E6%8B%93%E6%89%91.png)

- 配置OSPF，并抓包查看报文

```bash
[AR1]int lo 1
[AR1-LoopBack1]ip ad 1.1.1.1 24
[AR1-LoopBack1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip ad 12.1.1.1 24
[AR1-GigabitEthernet0/0/0]q
[AR1]ospf router-id 1.1.1.1
[AR1-ospf-1]area 0
[AR1-ospf-1-area-0.0.0.0]net 12.1.1.1 0.0.0.0
[AR1-ospf-1-area-0.0.0.0]net 1.1.1.1 0.0.0.0

[AR2]int lo 1
[AR2-LoopBack1]ip ad 2.2.2.2 32
[AR2-LoopBack1]int g0/0/0
[AR2-GigabitEthernet0/0/0]ip ad 12.1.1.2 24
[AR2-GigabitEthernet0/0/0]q
[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 12.1.1.2 0.0.0.0
```

- 报文分析

![OSPF发现并建立邻居_报文分析1](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5COSPF%E5%8F%91%E7%8E%B0%E5%B9%B6%E5%BB%BA%E7%AB%8B%E9%82%BB%E5%B1%85_%E6%8A%A5%E6%96%87%E5%88%86%E6%9E%901.png)

![OSPF发现并建立邻居_报文分析2](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5COSPF%E5%8F%91%E7%8E%B0%E5%B9%B6%E5%BB%BA%E7%AB%8B%E9%82%BB%E5%B1%85_%E6%8A%A5%E6%96%87%E5%88%86%E6%9E%902.png)

## 链路状态信息

- 链路状态信息LSA包含的信息：
	* 链路的类型
	* 接口IP地址及掩码
	* 链路上所连接的邻居路由器
	* 链路的带宽（开销）
- OSPF支持多种网络协议
	* **P2P**：PPP的网络类型。仅两台路由互联，支持广播和组播
	* **广播**：Ethernet-II的网络类型。两台或两台以上的路由器通过共享介质互联，支持广播和组播
	* **NBMA**：帧中继（FR）的网络类型。两台或两台以上的路由器通过VC互连，不支持广播和组播
	* **P2MP**：多个点到点的集合，支持广播、组播

**OSPF度量方式**

- 接口的cost=参考带宽/实际带宽
- 更改cost：
	* 直接在接口下改配置`ospf cost 10`(接口视图下)
	* 修改参考带宽（所有路由器都要修改，确保选路一致性）

> 计算路径开销累加是累加进接口的开销值

## 报文类型及作用

| Type | 报文名 | 功能 |
|:---:|:---|:---|
| 1 | Hello | 发现和维护邻居关系 |
| 2 | Database Description | 交互链路状态数据库摘要 |
| 3 | Link State Request | 请求特定的链路状态信息 |
| 4 | Link State Update | 发送详细的链路状态信息 |
| 5 | Link State Ack | 发送确认报文 |

## LSDB同步过程

![LSDB同步)](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5CLSDB%E5%90%8C%E6%AD%A5.png)

> - 第一份发出的DD包，不包含LSA的任何信息,用来选举主从
> 	* `I`位表示是否为第一个数据包
> 	* `M`位表示之后是否还有DD包
> 	* `MS`位为1表示为主,为0表示为从(通过比较Router-ID,大的为主)

![LSDB同步2](F:%5CGitHub%5CHCIP%20R&S%5COSPF.assets%5CLSDB%E5%90%8C%E6%AD%A52.png)

> - RTB中的路由信息如果RTA中没有,RTA会发送LSR请求报文,获取STB中的RTA没有的路由信息,RTA的路由信息RTB都有,RTB则直接进入Full
> - RTB收到RTA的请求报文,会把RTB中没有的路由信息放在LSU中,发送给RTA
> - RTA收到LSU后,更新自己的路由信息表,恢复LSAck报文确认
> - Full表示邻接关系建立成功

## DR与BDR的选举与作用

- 邻接关系个数=[n*(n-1)]/2

**DR与BDR的作用**

- 减少邻接关系
- 降低OSPF协议流量

**DR与BDR选举**

- DR与BDR的选举是基于接口
- 接口的DR优先级越大越优先
- 接口的DR优先级相等时,Router-ID越大越优先

> 更改网络类型`ospf network-type p2p`

# OSPF域内路由

## Router-LSA

**描述P2P网络**

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E4%B8%80%E7%B1%BBLSA%E6%8F%8F%E8%BF%B0P2P%E7%BD%91%E7%BB%9C.png" alt="一类LSA描述P2P网络" style="zoom:60%;" />

**描述MA网络或NBMA网络**

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E4%B8%80%E7%B1%BBLSA%E6%8F%8F%E8%BF%B0MA%E6%88%96NBMA%E7%BD%91%E7%BB%9C.png" alt="一类LSA描述MA或NBMA网络" style="zoom:60%;" />

## Network-LSA

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E4%BA%8C%E7%B1%BBLSA%E6%8F%8F%E8%BF%B0MA%E6%88%96NBMA%E7%BD%91%E7%BB%9C.png" alt="二类LSA描述MA或NBMA网络" style="zoom:60%;" />

> - `display ospf lsdb router`查看一类LSA
> - `display ospf routing`查看协议路由表

## SPF计算

- 构建SPF树
	* 根据Router-LSA和Network-LSA中的拓扑信息，构建SPF树干
- 计算最优路由
	* 基于SPF树干和Router-LSA、Network-LSA中的路由信息，计算最优路由

# OSPF域间路由

## 区域间路由传递

**Network-Summary-LSA**

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E4%B8%89%E7%B1%BBLSA%E5%86%85%E5%AE%B9.png" alt="三类LSA内容" style="zoom:60%;" />

> - 三类LSA是由ABR（边界路由器）产生的，用来描述区域间的路由信息
> - 三类LSA是在区域内泛洪，
> - 为本区域产生一条相邻区域的三类LSA

## 区域间路由防环

- 非骨干区域必须与骨干区域相连
- ABR收到非骨干区域的LSA不做计算
- 三类LSA传递规则

## 虚连接Vlink

- 进入OSPF进程`ospf 1`
- 进入中间区域`area 1`
- 虚连接`vlink-peer 对端的Router-ID`

# OSPF外部路由

## 外部路由引入

- 命令：`import-route static`

**AS-External-LSA**

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E4%BA%94%E7%B1%BBLSA%E5%86%85%E5%AE%B9.png" alt="五类LSA内容" style="zoom:60%;" />

> - ASBR产生，在整个OSPF域内泛洪
> - 引入外部路由，一条AS External LSA只能宣告一条外部路由

**ASBR-Summary-LSA**

<img src="F:%5CGitHub%5CHCIP%20R&amp;S%5COSPF.assets%5C%E5%9B%9B%E7%B1%BBLSA.png" alt="四类LSA" style="zoom:60%;" />

> - ABR产生，在区域内泛洪
> - 告诉其他设备如何去往ASBR
> - 通告到ASBR的开销

## 外部路由类型

| Type | Cost |
|:----:|:---|
| 第一类外部路由 | AS内部开销值+AS外部开销值 |
| 第二类外部路由 | AS外部开销 |

> - 默认为第二类外部路由
> - 第二类会先比较外部开销，外部开销相同，会比较内部开销
> - 第二类会可能会造成次优路径
> - External-Type1的优先级高于External-Type2（Type1比Type2更精确）

## 次优路径

**Forwarding Address**

- 字段为0
- 字段非0时，接口类型不能为P2P或P2MP接口、不能是静默接口、ASBR设备到达目的网段的下一跳地址所在网段被发布在OSPF中

> - **静默接口**：不再收发报文
> - 当三个条件都成立时，FA字段为非零，字段为ASBR到达目的网段的下一跳的地址

# OSPF特殊区域

## Stub区域和Totally Stub区域

- 传输区域
- 末端区域

**Stub区域**

- 配置：区域中的每台设备的OSPF进程下，进入区域，输入`stub`
- 不引入外部路由
- 没有四类、五类LSA
- 有一条三类的默认路由，有三类的明细路由

**Totally Stub区域**

- 配置：只需要在ABR上配置，进入OSPF进程，进入区域，输入`stub no-summary`
- 不能引入外部路由
- 有一条三类默认路由，没有三类明细、四类、五类LSA，

## NSSA区域和Totally NSSA区域

**NSSA区域**

- 允许外部路由
- 七类LSA只能在NSSA区域中传递
- 七类LSA传递给ABR，ABR转换为五类LSA，进行OSPF全区域泛洪
- 有三类的默认路由、明西路由

**Totally NSSA区域**

- 没有三类明细路由，有一条三类的默认路由

## 总结

| LSA类型 | 通告路由器 | LSA内容 | 传播范围 |
|:---:|:---:|:---|:---|
| Router LSA(Type-1) | OSPF Router | 拓扑信息、路由信息 | 本区域内 |
| Network LSA(Type-2) | DR | 拓扑信息、路由信息 | 本区域内 |
| Network-Summary-LSA(Type-3) | ABR | 域间路由信息 | 非Totally Stub区域(区域内) |
| ASBR-Summary-LSA(Type-4) | ABR | ASBR的Router ID以及到ASBR的开销 | 非特殊区域(区域内) |
| AS-External-LSA(Type-5) | ASBR | 路由进程域外部路由 | (非特殊区域)OSPF进程域 |
| NSSA LSA(Type-7) | ASBR | NSSA域外部路由信息 | (Totally)NSSA区域 |

## 区域间路由汇总和外部路由汇总

**区域间路由汇总**

- 配置：在边界路由器，进入OSPF进程，进入区域，输入`abr-summary 172.16.0.0 255.255.248.0`

**外部路由汇总**

- 配置：在ASBR上，进入OSPF进程，输入`bsbr-summary 172.17.0.0 255.255.248.0`