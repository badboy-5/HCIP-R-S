---
姓名：坏坏
学习时间：2020年5月5日
整理时间：2020年5月6日
---

参考博客[【CSDN】](https://blog.csdn.net/qq_45668124/article/details/105946789)

# STP

- 生成树STP（Spanning Tree Protocol）可以在提高可靠性的同时又能避免环路带来的各种问题
- 二层冗余带来的问题：
	* 广播风暴
	* MAC地址震荡

## STP的作用

- 通过阻塞逻辑端口来消除环路，并能够实现链路备份的目的
- 当可用路径发生故障时，STP可以自动激活备份链路，恢复网络的连通性

## STP操作

1. 选举一个根桥
2. 每个非根交换机选举一个根端口
3. 每个网段选举一个指定端口
4. 阻塞非根、非指定端口

### 选举根桥

- **根桥**是指Bridge-ID最小的交换机（BID最小的的交换机成为根网桥）
- Bridge-ID包括两部分（优先级、MAC地址）
- 修改优先级`stp priority 4096`(优先级必须是4096的倍数)

### 选举根端口

- 根端口指的是非根网桥到达根网桥路径开销最小的接口
- 沿着BPDU的流向（BPDU从根桥流向非根桥）累加入栈接口的开销值
- 接口开销值相同，则比较接口对端所连交换机的BID，BID小的交换机对应的接口为根端口
- 接口开销相同，对端BID相同，比较对端接口的Port-ID（优先级、接口编号）小的为根端口
- 上述所有参数都相同，则比较本地接口的Port-ID，小的为根端口

### 选举指定端口

- 在一个网段内到达根网桥路径开销最小的接口
- 根网桥上所有接口都是根端口
- 比较接口的根路径开销，根路径开销相同则比较此网段内所连交换机的BID，BID小的交换机的端口为指定端口
- 在两个交换机之间根据路径开销、对端BID、对端PID和本端的PID比较选举指定端口

	没有当选根端口和指定端口的端口会被阻塞

## 端口状态转换

- **Disabled**：生成树的禁用状态（端口关禁闭）
- **Blocking**：可以长期稳定的状态（阻塞）
- **Listening**：不稳定状态，只能收发BPDU，不能转发数据，学习MAC，等待15秒，切换状态
- **Learning**：不稳定状态，可以学习MAC表，不能转发数据
- **Forwarding**：稳定状态

## BPDU

- 包含桥ID、路径开销、端口ID、计时器等参数

## STP拓扑变化

- 根桥故障
	* 非根桥会在BPDU老化之后开始根桥的重新选举
- 根桥故障
	* 非根桥会在BPDU老化之后开始根桥的重新选举
- 直连链路故障
	* 检测到直连链路物理故障后，会将预备端口转换为根端口
	* 预备端口会在30s后恢复到转发状态
- 非直连链路故障
	* 预备端口恢复到转发状态大约需要50秒

### 拓扑变更导致MAC地址表变化

1. 当交换机检测到网络发生拓扑变化时，交换机会从根端口向外发出TCN BPDU
2. 上游交换机收到TCN BPDU后，会以TCA位被置位的BPDU进行应答
3. 上游交换机收到TCN BPDU后，会进一步从根端口转发该BPDU ，直到TCN BPDU抵达根网桥
4. 根网桥收到TCN BPDU后，会全网泛洪TC位被置位配置BPDU
5. 交换机收到TC位被置位的配置BPDU后，触发生成树的重新计算
6. 同时收到TC BPDU的端口的MAC地址表老化时间缩短至转发延迟

## STP模式

- STP配置：`stp mode stp`(华为设备默认工作模式为MSTP)
- 配置路径开销值：接口视图下`stp cost 2000`

# RSTP

- 传统STP收敛速度慢，至少需要30S
- 根桥直连链路down掉，BP端口换成DP端口并进入转发状态需要大约50S
- 交换机连接终端的链路进入转发需要经过30S
- 拓扑变更机制复杂，效率低下
- 端口角色

> - Disabled、Blocking、Listening都不转发用户流量也不学习MAC地址
> - Learning不转发用户流量，但是学习MAC地址表
> - Forwarding既转发用户流量又学习MAC地址

## RSTP优点

### 端口角色重新划分

- 在STP原有的根端口和指定端口上新定义了新的端口
- Backup Port（BP指定端口的备份）
- Alternate Port（AP根端口的备份）

### 端口状态重新划分

- 由原来STP的5种状态缩减为3种状态
- STP的Disabled、Blocking、Listening对应RSTP的Discarding状态（不转发用户流量也不学习MAC地址）
- STP的Learning、Forwarding在RSTP中不变

<table>
	<tr>
		<caption>
			<th>STP端口状态</th>
			<th>RSTP端口状态</th>
			<th>端口状态对应行为</th>
		</caption>
	</tr>
	<tr >
	    <td>Disabled</td>
	    <td rowspan="3">Discarding</td>
	    <td rowspan="3">不转发用户流量也不学习MAC</td>
	</tr>
	<tr >
	    <td>Blocking</td>
	</tr>
	<tr >
	    <td>Listening</td>
	</tr>
	<tr >
	    <td>Learning</td>
	    <td>Learning</td>
	    <td>不转发用户流量，但是学习MAC地址</td>
	</tr>
	<tr >
	    <td>Forwarding</td>
	    <td>Forwarding</td>
	    <td>既转发用户流量又学习MAC地址</td>
	</tr>
</table>

## 快速收敛机制

### P/A机制

- 对指定端口（DP）快速进入转发状态生效
- DP向下游交换机发出P位被置位的BPDU
- 下游交换机接收到P位被置位的BPDU后，首先会执行Sync的操作（交换机会阻塞所有处于转发状态的非边缘端口）
- 然后下游交换机会向上游交换机发送A位被置位的BPDU
- 上游交换机收到A位被置位的BPDU后，立刻将DP口置为转发状态，无需等待定时器到期

> - Proposal/Agreement机制，其目的是使一个指定端口尽快进入Forwarding状态
> - P/A机制要求两台交换设备之间的链路必须是点对点的全双工模式
> - 一旦P/A机制协商不成功，指定端口的选择就需要等待两个Forward Delay（30s），协商过程与STP一样

![P-A机制](F:\GitHub\HCIP R&S\STP、RSTP与MSTP.assets\P-A机制.png)

**阶段一**

- 设备刚启动，都认为自己是根桥，向外发送P置位的BPDU，把发送BPDU的接口变成DP口，接口处于Discarding状态

**阶段二**

- SWA的优先级最高，所以不会理会SWB和SWC的BPDU
- SWB、SWC收到SWA的BPDU后，共同认为SWA为最优根桥，根据P/A协商，回复A置位的报文，并把发送端口变为RP端口，接口处于Forwarding状态

**阶段三**

- SWB与SWA互相交换自己认为最优的BPDU（SWA的BPDU）
- 彼此收到对方的最优BPDU后，发现根一致，但SWB的优先级高于SWC的优先级，所以SWC停止发送P置位的报文，且以有端口为RP，所以不会回复A置位的报文
- SWB则会一直给SWC发送P置位的报文，等待两个Forward Delay时间（30s）后，SWB面向SWC的接口为DP端口，处于Forwarding状态，SWC面向SWB的接口为AP端口，处于Discarding状态

### 根端口快速切换机制

- 当原有的根端口发生故障，且AP口对端连接的DP口处于转发状态，AP口立刻转换为RP口，并进入转发状态

### 次等BPDU处理机制

- 发生故障的交换机以自己为根桥，向外发送P位被置位的BPDU，对端收到后，会将本地最优的BPDU回给故障交换机
- 发生故障的交换机收到最优的BPDU后，向对端发送A位被置位的BPDU
- 对端收到A位被置位的BPDU后，立刻将AP口转换为DP口，进入转发状态

![次等BPDU](F:\GitHub\HCIP R&S\STP、RSTP与MSTP.assets\次等BPDU.png)

- 当SWB的上行链路Down后，SWB认为自己是根桥，向SWC发送P置位的BPDU
- SWC收到SWB的BPDU后，发现本地的BPDU比SWB的更优，会把本地的最优BPDU发送给SWB，同时端口为从AR变为DP，处于Forwarding状态
- SWB收到最优的BPDU后，重新定义端口角色，将DP更改为RP，然后发送A置位的BPDU

## 边缘端口的引入

- 交换机的接口被设置为边缘端口（EP），则改端口不参与生成树的运算，一旦接入，立刻进入转发状态

> - 自动选举只选举DP、RP、AP、BP，边缘端口EP需要手动指定
> - 边缘端口UP后，立马进入转发状态
> - P/A机制时，不会被同步
> - 边缘端口会周期性（2s）向外发送BPDU
> - 边缘端口收到BPDU后，会失去边缘端口的特性，重新参与生成树的选举
> - 边缘端口迁移到Forwarding状态不会导致拓扑变化

## 拓扑变更机制的优化

- 判断拓扑变化唯一标准：一个非边缘端口迁移到Forwarding状态
- 变更点直接产生TC BPDU触发生成树的重新运算，RSTP不再使用TCN BPDU
- TC BPDU会直接删除错误的MAC地址表项，不再将MAC地址表项的老化时间降低为转发延迟，进一步提高收敛速度

## BPDU保护

- 边缘端口收到BPDU后，自动将其设置为非边缘端口，重新进行生成树的计算
- **BPDU保护**边缘端口配置BPDU保护，边缘端口收到BPDU后，交换机立刻关闭该边缘端口

## 根保护

- 配置了根保护的接口如果收到更优的BPDU，则进入Discarding状态，不再转发该BPDU
- 一段时间内收不到更优的BPDU，则改端口进入转发状态，保护根桥位置

## TC-BPDU泛洪保护

- 如果交换机短时间收到大量的TC-BPDU，交换机会频繁的执行MAC地址表的删除操作和生成树的重新运算，给设备增加负担
- 交换机配置了TC-BPDU保护后，交换机只会处理单位时间内人为设置的TC-BPDU的数量，超出部分交换机不处理

## RSTP配置

```bash
# 系统视图下
stp mode rstp  //将交换机的STP类型更改为RSTP
stp root primary  //设置为根桥
stp bpdu-protection  //开启BPDU保护功能

# 接口视图下
stp edged-port enable  //将当前接口设置为边缘端口
stp root-protection  //在当前接口开启根保护

stp tc-protection  //开启TC-BPDU保护功能
stp tc-protection threshold 5  //设置单位时间内处理TC-BPDU数量的阀值
```

# MSTP

- RSTP在STP的基础上进行了改进，实现了网络拓扑快速收敛，但是由于局域网内所有的VLAN共享一棵生成树，因此被阻塞后，链路将不承载任何流量，无法在VLAN间实现数据流量的负载均衡，从而造成带宽浪费
- 单生成树的弊端
	* 部分VLAN路径不通
	* 无法实现流量分担
	* 次优二层路径

## 多生成树实例解决单生成树弊端

- 多生成树协议MSTP（Multiple Spanning Tree Protocol）
- MST域是多生成树域（Multiple Spanning Tree Region），由交换网络中的多台交换设备以及他们之间的网段所构成
- 每个MST域内可以有多棵生成树，每棵生成树为一个MSTI。MSTI之间彼此独立，每个MSTI的计算过程基本与RTSP的计算过程相同

## MSTP配置

**实验**：如下拓扑图

![MSTP实验拓扑](F:\GitHub\HCIP R&S\STP、RSTP与MSTP.assets\MSTP实验拓扑.png)

- 创建VLAN

```bash
[SW1]vlan batch 10 20 30
[SW2]vlan batch 10 20 30
[SW3]vlan batch 10 20 30
```

> stp模式默认为MSTP，`stp mode mstp`修改为MSTP

- 配置MSTP

```bash
[SW1]stp region-configuration  //进入预配置
[SW1-mst-region]region-name bad  //配置域名
[SW1-mst-region]instance 1 vlan 10 20  //创建实例1，允许vlan 10 20
[SW1-mst-region]instance 2 vlan 30  //创建实例2，允许vlan 30
[SW1-mst-region]active region-configuration  //激活预配值
```

> -  必须激活预配值才能生效
> - SW2、SW3与SW1相同的配置
> - 三台设备如果要配置搭配同一个域，需要把域名配置相同

- 配置不同实例的根桥

```bash
[SW2]stp instance 1 root primary  //强制配置SW2为实例1的根桥
[SW2]stp instance 2 root secondary  //指定SW2为实例2的备份根桥
```

```bash
[SW3]stp instance 2 root primary  //强制配置SW3为实例2的根桥
[SW3]stp instance 1 root secondary  //指定SW3为实例1的备份根桥
```

> - 如果不指定根桥，则会根据RSTP自动选举根桥
> - 可以通过强制指定根桥或者更改优先级改变根桥

- 配置链路为Trunk链路

```bash
[SW3]int e0/0/1
[SW3-Ethernet0/0/1]port link-type trunk 
[SW3-Ethernet0/0/1]port trunk allow-pass vlan all
[SW3-Ethernet0/0/1]int e0/0/2
[SW3-Ethernet0/0/2]port link-type trunk 
[SW3-Ethernet0/0/2]port trunk allow-pass vlan all 
```

> - 每台交换机之间的链路都必须设置成trunk链路，允许vlan通过
> - 设置完成后`dis stp bri`查看stp生成树