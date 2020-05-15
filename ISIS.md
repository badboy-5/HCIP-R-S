---
姓名：坏坏
学习时间：2020年5月14日
整理时间：2020年5月14日
---

# IS-IS协议基本原理

- IS-IS工作在数据链路层，支持CLNP和IP
- OSPF工作在网络层，仅支持IP

**路由计算**

- 建立邻居关系
- 同步LSDB数据库
- 执行SPF路由计算

**地址结构**

| TCP/IP | IP协议 | IP地址 | OSPF |Area ID+Router ID |
|:---:|:---:|:---:|:---:|:---:|
| OSI系统 | CLNP协议 | NSAP地址 | IS-IS | NET标识符 |

## NSAP报文

![NSAP](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CNSAP.png)

> - IDP相当于IP地址中的主网络号，由AFI与IDI两部分组成
> 	* AFI表示地址分配机构和地址格式，IDI用来标识域
> - DSP相当于IP地址中的子网号和主机地址，由High Order DSP、System ID和SEL三个部分组成
> 	* High Order DSP用来分割区域
> 	* System ID用来区分主机
> 	* SEL用来指示服务类型
> - Area ID能够标识路由域，也能够标识路由域中的区域，相当于OSPF中的区域编号
> - System ID用来在区域内唯一标识主机或路由器
> - SEL表示不同的传输协议，在IP上SEL均为00

## 路由器分类

- IS-IS路由器的三种类型
	* Level-１路由器（只能创建level-1的LSDB）
	* Level-2路由器（只能创建level-2的LSDB）
	* Level-1-2路由器（路由器的默认类型，能同时创建level-1和level-2的LSDB）

> - Level-1的邻接关系的建立前提是区域ID必须一致
> - Level-2可以是不同区域
> - IS-IS支持点对点和广播网络类型

## 邻居关系建立

- P2P网络邻居关系建立可以是两次握手也可以是三次握手
- MA网络邻居关系建立必须是三次握手

![P2P两次握手](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CP2P%E4%B8%A4%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

- 两次握手只要路由器收到对端发来的Hello报文，就单方面宣布邻居为up状态，建立邻居关系，不过容易存在单通风险

> IIH即IS-IS Hello

![P2P&MA三次握手](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CP2P&MA%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

- 当收到邻居发送的Hello报文里面没有自己的MAC的时候，状态机进入initialized
- 只有收到邻居发过来的Hello报文中有自己的MAC才会up，排除了链路单通的风险
- UP之后会选举DIS（指定中间系统——Designated IS） 

## DIS与DR类比

| 类比点 | ISIS-DIS | OSPF-DR |
|:---:|:---|:---|
| 选举优先级 | 所有优先级都参与选举 | 0优先级不参与选举 |
| 选举等待时间 | 2个Hello报文间隔（20S） | 40S或120S |
| 备份 | 无备份 | 有备份（BDR） |
| 邻接关系 | 所有路由器互相都是邻接关系 | DRother之间是2-way关系 |
| 抢占性 | 会抢占 | 不会抢占 |
| 作用 | 周期发送CSNP，保障MA网络LSDB同步 | 主要为了减少LSA泛洪 |

> OSPF存在网络抖动，IS-IS不存在抖动，所以IS-IS的DIS没有备份

## 链路状态信息的载体

- LSP PDU——用于交换链路状态信息
	* 实节点LSP
	* 伪节点LSP（只在广播链路存在）
- SNP PDU——用于维护LSDB的完整与同步，且为摘要信息
	* CSNP（用于同步LSP）
	* PSNP（用于请求和确认LSP）

> - 在MA网络中所有协议报文的目的MAC地址都是组播地址
> 	* Level-1：0180-C200-0014
> 	* Level-2：0180-C200-0015

**TLV**

- 可以通过加减TLV，增加删除功能

![TLV](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CTLV.png)

## 链路状态信息交互

![P2P中交互LSP](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CP2P%E4%B8%AD%E4%BA%A4%E4%BA%92LSP.png)

> - 邻居建立后，RTA发送CSNP报文，包含此设备的所有的路由信息
> - RTB收到CSNP报文后，查看自己的LSDB，将RTA上自己没有的路由信息包装在PSNP报文中，发送给RTA
> - RTA收到后，将含有请求同步的LSP信息发给RTB，RTA发送LSP报文后，5秒内会等待RTB的回复，如果没有收到RTB的回复确认，五秒后会再次发送LSP报文
> - RTB收到RTA的LSP报文后，会回复PSNP报文，确认收到RTA的报文
> - P2P网络CSNP报文只发送一次，邻居建立后立即发送

**实验**：拓扑如下

![实验1拓扑图](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5C%E5%AE%9E%E9%AA%8C1%E6%8B%93%E6%89%91%E5%9B%BE.png)

- 基础配置

```bash
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip ad 12.1.1.1 24

[AR2]int g0/0/0
[AR2-GigabitEthernet0/0/0]ip ad 12.1.1.2 24
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]ip ad 23.1.1.2 24 

[AR3]int g0/0/0
[AR3-GigabitEthernet0/0/0]ip ad 23.1.1.3 24
[AR3]int lo 0
[AR3-LoopBack0]ip ad 3.3.3.3 32
```

- 配置IS-IS

```bash
[AR1]isis  //起IS-IS进程
[AR1-isis-1]network-entity 49.0001.0000.0000.0001.00  //配置net
[AR1-isis-1]q
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]isis enable 1  //宣告接口

[AR2]isis 1
[AR2-isis-1]network-entity 49.0001.0000.0000.0002.00  //配置net
[AR2-isis-1]int g0/0/0
[AR2-GigabitEthernet0/0/0]isis enable 1  //宣告接口
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]isis enable 1

[AR3]isis 1
[AR3-isis-1]network-entity 49.0002.0000.0000.0003.00
[AR3-isis-1]int g0/0/0
[AR3-GigabitEthernet0/0/0]isis enable 1

[AR1]dis isis peer  //查看ISIS状态
```

**AR1上IS-IS邻居**

![AR1上IS-IS邻居](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CAR1%E4%B8%8AIS-IS%E9%82%BB%E5%B1%85.png)

**AR3上IS-IS邻居**

![AR3上IS-IS邻居](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CAR3%E4%B8%8AIS-IS%E9%82%BB%E5%B1%85.png)

**AR1上IS-IS学习到的路由**

![AR1上IS-IS学习的路由](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CAR1%E4%B8%8AIS-IS%E5%AD%A6%E4%B9%A0%E7%9A%84%E8%B7%AF%E7%94%B1.png)

**详细LSP内容**

![AR3的ISIS详细信息](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CAR3%E7%9A%84ISIS%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF.png)

**DIS的判断**

![DIS](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CDIS.png)

**抓包分析**

![IS-IS报文分析](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CIS-IS%E6%8A%A5%E6%96%87%E5%88%86%E6%9E%90.png)

![IS-IS的LSP报文报文](F:%5CGitHub%5CHCIP%20R&S%5CISIS.assets%5CIS-IS%E7%9A%84LSP%E6%8A%A5%E6%96%87%E6%8A%A5%E6%96%87.png)

**拓展**

```bash
# 修改IS-IS优先级（默认为64）
[AR2]int g0/0/0 
[AR2-GigabitEthernet0/0/0]isis dis-priority 70  //修改优先级，使R2成为DIS
# 修改链路类型两端的设备都需要更改
[AR2-GigabitEthernet0/0/0]isis circuit-type p2p  //修改链路类型为P2P没有DIS
```

## 路由算法

- ISIS路由计算开销方式：
	* 设备默认接口开销值都是10
- SPF计算过程：
	* 单区域LSDB同步完成
	* 生成全网拓扑结构图
	* 以本节点为根，生成最短路径树
	* 默认跨越每个节点开销一样

## 网络分层路由域

- **Backbone**是由L2+L1/L2组合的区域构成的骨干区域
- 区域号对区域的划分无作用
- IS-IS的区域边界在整个路由器，OSPF的边界在接口

## 区域间路由

- 非骨干区域会向骨干区域发送LSP，默认情况下骨干区域不会向非骨干区域发送LSP
- 非骨干区域的路由器会向L1/L2发送ATT被置位的LSP，收到的L1/L2会产生已调配默认路由
- 非骨干区域访问骨干区域的路由器，通过默认路由访问，骨干区域路由器访问非骨干区域路由器可以通过明细路由

**路由泄露**

```bash
[AR2]isis 
[AR2-isis-1]import-route isis level-2 into level-1  //level-2引入level-1
```

# IS-IS与OSPF的区别

| 差异性 | IS-IS | OSPF |
|:---:|:---:|:---:|
| 网络类型 | 少 | 多 |
| 开销方式 | 复杂 | 简便 |
| 区域类型 | 少 | 多 |
| 路由报文类型 | 简单 | 多样 |
| 路由收敛速度 | 很快 | 快 |
| 拓展性 | 强 | 一般 |
| 路由负载能力 | 超强 | 强 |

# IS-IS应用场景配置

- 园区网：OSPF
	* 区域多变、策略多变、调度精细
- 骨干网：IS-IS、BGP
	* 区域扁平、收敛极快、承载庞大

> IGP：内部网关协议
> EGP：外部网关协议