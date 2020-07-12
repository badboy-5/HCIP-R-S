---
姓名：坏坏
学习时间：2020年6月19日
整理时间：2020年6月19日
---

# PIM协议

- PIM（Protocol Independent Multicast）协议无关组播，PIM直接利用单播路由表的路由的路由信息进行组播报文RPF检查，创建组播路由表项，转发组播报文

# PIM—DM

## PIM-DM基本概述

- DM密集模式
- 采用“推（Push）模式”转发组播报文
- PIM-DM的关键任务：
	* 建立SPT（Shortest Path Tree）
- 组播地址：224.0.0.13

> - 推（Push）模式：无论是否有主机需要该组播报文，都将转发

## PIM-DM邻居发现

- 使用Hello机制发现邻居
- 选举DR
	* 优先级高的成为DR
	* IP地址大的成为DR

## PIM-DM构建SPT

- 扩散
- RPF检查
- 剪枝

1. 组播源向组播组发送数据报文，组播组内的所有设备都会继续向下转发数据报文（扩散）
2. 当有路径不需要该组播的数据，最后一跳路由器会向上发送Prune
3. 上游设备收到后，将不会再向这条下游设备转发该组的数据（剪枝）
4. RPF检查：当设备从多个接口收到同一份组播数据以后，会根据RIB（本地路由表）检查到达这份数据的源怎么走，从而选择相应接口的数据报文

> 剪枝计时器210s，剪枝已知计时器3s

## 状态刷新（SR）

- 周期性的刷新剪枝端口状态（60s）

## 嫁接（Graft机制）

- 新的组成员加入组播组后，快速得到组播报文

## 断言（Assert机制）

- 避免重复组报文

1. 当多台设备同时向一个设备发送相同的组播报文时，多台设备都会发送断言报文
2. 通过比较RIB中学习的路由条目，选择单播路由协议优先级高的
3. 各设备到组播源的开销，开销小的优先
4. 下游接口IP地址最大的优先

> 没有被选为转发数据的设备，180s会重新转发一次数据

**PIM-DM配置**

- 运行了IGMP后，直接在接口下使用`pim dm`配置

```bash
[R1]int g0/0/1
[R1-GigabitEthernet0/0/1]pim dm
```

> - `dis pim neighbor`查看pim邻居建立
> - `dis multicast routing-table`查看组播路由表

> - PIM-DM适用于组播成员分布较为密集的园区网络
> - PIM-DM在组播成员分布较为稀疏的网络中，组播流量的周期性扩散会给网络带来较大负担

[【PIM-DM实验】](https://blog.csdn.net/qq_45668124/article/details/106873627)

# PIM—SM

## PIM-SM基本概述

- SM稀疏模式
- 使用“拉（Pull）模式”转发组播报文
- PIM-SM的关键任务：
	* 建立RPT（Rendezvous Point Tree，汇聚点树，也叫共享树）
	* 建立SPT（Shortest Path Tree，最短路径树）
- 适用于组播成员分布较为稀疏的网络环境

## 汇聚点RP（Rendezvous Point）

- 充当RPT树的根节点
- 共享树中的所有组播流量都经过RP转发给接收者
- 所有PIM路由器都要知道RP的位置

## RPT建立

- 主机加入某个组播时，发送IGMP成员关系报告
- 最后一跳路由器向RP发送（*，G）Join
- （*，G）Join报文到达RP的过程中，沿途各路由器都会生成响应的（*，G）组播转发条目

> RPT实现了组播数据按需转发的目的，减少了数据泛洪对网络带宽的占用

## 接收者侧DR和组播源侧DR

- 运行PIM-SM的网络，都会进行DR的选举
- 组播接收者侧DR：与组播组成员相连的DR，负责向RP发送（*，G）的Join加入消息
- 组播源侧DR：与组播源相连的DR，负责向RP发送单播的Register报文

> PIM-SM中DR的选举原则与PIM-DM相同

## SPT建立

- 组播源向组播组发送第一个组播报文
- 源端DR将该组播报文封装成Register报文，并以单播方式发送给响应的RP
- RP接收到Register报文后，从Register报文中提取出组播报文，将该组播报文沿RPT分支发送给接收者
- SPT树建立后，组播源发出的组播报文沿着SPT转发至RP
- RP沿SPT收到该组播报文后，向源端DR单播发送Register-Stop报文，停止注册

<table>
	<tr>
		<th>模式</th>
		<th>类型</th>
		<th>使用场景</th>
	</tr>
	<tr >
	    <td>PIM-DM</td>
	    <td>(S，G)</td>
	    <td>第一跳路由器到最后一跳路由器的SPT</td>
	</tr>
	<tr >
	    <td rowspan="3">PIM-SM</td>
	    <td>(*，G)</td>
	    <td>RP到最后一跳路由器的RPT</td>
	</tr>
	<tr >
	    <td>(S，G)</td>
	    <td>源端DR到RP的SPT</td>
	</tr>
	<tr >
	    <td>(S，G)</td>
	    <td>Switchover之后，从第一跳路由器到最后一跳路由器的SPT</td>
	</tr>
</table>

## PIM-SM转发树

- 组播源发出的组播报文沿着SPT到达RP，从RP沿RPT到达接收者
- 从组播源到接收者的路径不一定最优，且RP的工作负担大

## Switchover机制

- 切树
- 用户端DR周期性检测组播报文的转发速率，当速率超过阈值（默认为0），则会触发SPT切换
	* 用户端DR逐跳向源DR发送（S，G）Join报文，并创建（S，G）表项，建立源端DR到用户端DR的SPT
	* SPT建立后，用户端DR沿RPT逐跳向RP发送剪枝报文，收到剪枝报文的路由器将（*，G）复制成相应的（S，G），并将相应的下游接口置为剪枝状态。剪枝结束后，RP不再沿RPT转发组播报文到组成员
	* 如果SPT不经过RP，RP会继续向源端DR逐跳发送剪枝报文，删除（S，G）表项中相应的下游接口。剪枝结束后，源端DR不再向RP转发组播报文

[【组播综合实验】](https://blog.csdn.net/qq_45668124/article/details/106879488)