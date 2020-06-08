---
姓名：坏坏
学习时间：2020年5月28日
整理时间：2020年6月8日
---

# 复习

**IGP协议**（内部网关协议）

- RIP（已淘汰）
- OSPF
- IS-IS

**EGP协议**（外部网关协议）
- BGP

> - IGP用来计算和发现路由，BGP路由控制和路由优选（13条选路原则）
> - IGP是在自治系统（AS）内部使用，EGP是在自治系统间使用

# BGP基本概述

- BGP可以跨越多跳路由建立邻居关系
- 通过单播发送报文
- 基于TCP，端口号179，为应用层协议

# BGP邻居关系建立与配置

- BGP基于TCP，所以建立邻居前会先建立三次握手
- 先启动BGP的一端先发起TCP连接

## BGP邻居类型

**EBGP**

- 外部边界网关协议
- 运行在不同的AS之间的BGP路由器建立的邻居关系叫EBGP（External BGP）邻居关系

**IBGP**

- 内部边界网关协议
- 运行在相同AS内部的BGP路由器建立的邻居关系为IBGP（Internal BGP）邻居关系
- 在AS内部，不使单独用BGP协议，通常是IGP+BGP

## BGP配置

- `router-id 1.1.1.1`指定router-id
- `bgp 300`进入到自身的AS区域
- `peer 10.1.1.3 as-number 100`对方的IP和AS区域
- `peer 2.2.2.2 connect-interface loopback 0`指定更新源，更稳定

> - BGP是在多个站点之间传递路由，并不是为了在AS内部打通路由
> - EBGP之间建邻居一般使用直连接口
> - IBGP之间建邻居一般使用环回口

[【BGP练习实验】](https://blog.csdn.net/qq_45668124/article/details/106475666)

## BGP邻居关系建立

- BGP报文格式
	* BGP报文头部
	* OPEN报文，建立连接
	* UPDATE报文，更新路由
	* NOTIFICATION报文，通知错误
	* KEEPALIVE报文，维护TCP的连接
	* REFRESH报文，请求得到路由

![BGP邻居建立](F:%5CGitHub%5CHCIP%20R&S%5CBGP.assets%5CBGP%E9%82%BB%E5%B1%85%E5%BB%BA%E7%AB%8B.png)

> - 先建立TCP连接
> - TCP连接建立后，发送`Open`报文，另一端接收到报文后，会进行检验
> - 检验符合，回复`Keepalive`，邻居建立成功
> - `Keepalive`60s发送一次，180s收不到就认为对端down了
> - `Update`路由发生变化时，会发送`Update`报文

## BGP状态机

![BGP状态机](F:%5CGitHub%5CHCIP%20R&S%5CBGP.assets%5CBGP%E7%8A%B6%E6%80%81%E6%9C%BA.png)

- `IDLE`等待触发邻居建立的事件
- `Connect`配置好BGP后，开始建立TCP连接
	* TCP连接建立成功，进入`Open-sent`状态
	* TCP连接建立失败，进入`Active`状态，重新建立TCP连接
- open-sent向外发送OPEN报文，收到正确的报文，进入`Open-confirm`状态
- 接收到Keepalive报文，进入`Established`状态，邻居建立成功

# BGP路由生成方式

- Network，只能宣告路由表中有的路由，且与路由表中的路由完全相同
- import，根据运行的路由协议，将路由引入到BGP路由表中，还可以引入直连和静态路由

# BGP通告原则和路由处理

- BGP通过Network和import两种方式生成路由，BGP邻居关系建立后，开始将路由封装在Update报文中通告给邻居
- Update报文主要用来通告可用路由和撤销路由，Update包含：
	* 网络层可达信息（NLRI）：公布IP前缀和前缀长度，发布路由
	* 路径属性：为BGP提供环路检测，控制路由优先
	* 撤销路由：描述无法到达且从业务中撤销的路由前缀和前缀长度

## BGP通告原则

- 仅将自己最优的路由发布给邻居
- 通过EBGP获得的最优路由发布给所有的BGP邻居
- 通过IBGP获得的最优路由不会发布给其他的IBGP邻居（水平分割，防环）
- BGP与IGP同步（默认关闭）

## BGP路由信息处理

![BGP路由信息处理](F:%5CGitHub%5CHCIP%20R&S%5CBGP.assets%5CBGP%E8%B7%AF%E7%94%B1%E4%BF%A1%E6%81%AF%E5%A4%84%E7%90%86.png)

- 从BGP邻居收到更新信息后，进行路径选择
- 将路由信息，写入本地的路由表
- 将BGP路由表中最优的路由下发到全局路由表，为后面的数据转发做准备
- 把本地的最优路由发布给其它邻居

> 在发布最优路由之前，如果接口有策略，需要先过策略，再转发给BGP邻居

# BGP常用属性介绍

![BGP属性](F:%5CGitHub%5CHCIP%20R&S%5CBGP.assets%5CBGP%E5%B1%9E%E6%80%A7.png)

- **公认必遵**：公共认可，并且三个属性必须存放在Update报文中
	* **Origin**（起源，初始）：定义路径信息的来源。`i`是network引入的，`?`是import引入的，`e`是EGP引入的
	* **AS_Path**（AS路径）：标识AS区域号，每经过一个AS区域，AS号会附加。防止环路
	* **Next_hop**(下一跳)：路由传给IBGP邻居时下一跳不变，路由传给EBGP邻居时，下一跳为和对方建邻居的地址

- **公认任意**：公共认可，可有可无的属性
	* **Local_Prefference**：本地优先级。在AS内，IBGP之间传递路由，默认本地优先级为100，越大越优，影响本AS出去的流量
	* **MED**：在两个AS间，EBGP之间传递路由，默认值为0，越小越优，影响本AS进来的流量，且只能传递一个AS，用于判断流量进入AS时的最佳路由

- **可选过渡**：无论是否可以识别报文信息，都必须转发
	* **Community**：团体属性。表示路由，限定路由的传播范围，打标记，便于对符合相同条件的路由进行统一处理

> - 公认团体属性：
> 	* `Internet`：缺省属性，此路由可以通告给左右的BGP邻居
> 	* `No_Export`：不将此路由发布到其他AS
> 	* `No_Advertise`：不将此路由通告给其他的BGP邻居
> 	* `No_Export_Subconfed`：联盟中使用
> - 拓展的团体属性：

- **可选非过渡**：不管是否可以识别报文信息，可以转发也可以不转发

> - BGP防环：
> 	* AS内防环：通过IBGP获得的最优路由不会发布给其他的IBGP邻居（水平分割）
> 	* AS间防环：AS_Path增加AS区域号

# BGP选路原则

- BGP路由器将路由通告给邻居后，每个BGP邻居都会进行路由优选，路由选择有三种情况：
	* 该路由是到达目的地的唯一路由，直接优选
	* 对到达同意目的地的多条路由，优选优先级最高的
	* 对到达同同一目的地且具有相同优先级的多条路由，必须用更新的原则去选择一条最优的

## BGP13条选路原则

- BGP计算路由优先级的13条规则：
	1. 丢弃下一跳不可达的路由
	2. 优选Preference_Value值最高的路由（私有属性，仅本地有效）---------P
	3. 优选本地优先级（Local_Preference）最高的路由--------------------L
	4. 优选手动聚合->自动聚合->network->import->从对等体学到的---------L
	5. 优选AS_Path短的路由-------------------------------------------A
	6. 起源类型IGP->EGP->Incomplete----------------------------------O
	7. 对于来自同一AS的路由，优选MED值小的-----------------------------M
	8. 优选从EBGP学来的路由（EBGP>IBGP）------------------------------E
	9. 优选AS内部下一跳的IGP的Metric最小的路由-------------------------N
	10. 优选Cluster_List最短的路由
	11. 优选OrGinator_ID最小的路由
	12. 优选Router_ID最小的路由器发布的路由
	13. 优选具有较小IP地址的邻居学来的路由

> - `PLLAOMEN`漂亮老男人
> - `P`Preference_Value
> - `L`Local_Preference，本地优先级
> - `L`手动聚合->自动聚合->network->import，为本地始发
> - `A`AS_Path
> - `O`Origin，起源
> - `M`MED
> - `E`EBGP优于IBGP
> - `N`Next-Hop，下一跳

> 前8条选路原则完全相同，最大负载条目大于等于2，就不会再向后比，直接负载

## 各种属性对选路的影响

**Preference_Value**

- Preference_Value是BGP的私有属性，Preference_Value相当于BGP选路规则中Weight值，仅在本地路由器生效，Preference_Value值越大，越优先

**聚合**

- 聚合路由优先级：手动聚合>自动聚合

**EBGP邻居的路由优于IBGP邻居的路由**

- AS内的路由器收到IBGP和EBGP的路由，会优先选择EBGP学习到的路由

**AS内部IGP的Metric**

- 调整IGP的Cost，提升带宽

[路由策略详细介绍](https://blog.csdn.net/qq_45668124/article/details/106627507)

# BGP路由聚合

- BGP在AS之间传递路由信息，AS的数量增多，单个AS规模扩大，BGP路由表变大，带来的问题：
	* 存储路由表占用大量内存资源，传输和处理路由信息消耗大量贷款
	* 传输的路由条目出现频繁的更新和撤销，对网络的稳定性造成影响

## BGP路由聚合的必要性

- 将以下路由聚合
	* `10.1.8.0/24`
	* `10.1.9.0/24`
	* `10.1.10.0/24`
	* `10.1.11.0/24`

**静态**

- 静态一般用的较少

```bash
ip route-static 10.1.8.0 22 null 0
network 10.1.8.0 22
```

**自动聚合**

```bash
summary automatic  //自动聚合，按主类聚合，掩码为8
```

**手动聚合**

[【聚合实验及详细说明】](https://blog.csdn.net/qq_45668124/article/details/106627326)