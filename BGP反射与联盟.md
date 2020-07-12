---
姓名：坏坏
学习时间：2020年6月16日
整理时间：2020年6月16日
---

# 反射

- 路由反射器RR（Router Reflector）
	* 允许把从IBGP对等体学到的路由反射到其他IBGP对等体的IBGP设备，类似OSPF网络中的DR
- 客户机（Client）
	* 与RR形成反射邻居关系的IBGP设备，在AS内部，客户机只需要与RR直连
- 非客户机（Non-Client）
	* 即便是RR也不是客户机的IBGP设备。在AS内部非客户机与RR之间，以及所有的非客户机之间仍然必须建立全连接关系
- 始发者（Originator）
	* 在AS内部事发路由的设备。Originator_ID属性用于防止集群内产生路由环路
- 集群（Cluster）
	* 路由反射器及其客户机的集合。Cluster_List属性用于防止集群间产生路由环路

## 反射器规则

- RR从EBGP收到的路由，即会反射给客户端，也会反射给非客户端
- RR从客户端收到的路由会反射给客户端
- RR从非客户端收到路由只会反射给客户端和EBGP邻居，不会反射给其他的非客户端

```bash
[R1-bgp]peer 10.1.12.2 reflect-client  //指定客户机
```


# 联盟

**实验**：如下拓扑图

![1592297418580](F:\GitHub\HCIP R&S\BGP反射与联盟.assets\1592297418580.png)

## 基本配置

```bash
# AR1基础配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
```

```bash
# AR2基础配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
```

```bash
# AR3基础配置
 sysname AR3
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.34.3 255.255.255.0 
```

```bash
# AR4基础配置
 sysname AR4
#
interface GigabitEthernet0/0/0
 ip address 10.1.34.4 255.255.255.0
```

## 配置联盟

```bash
[AR2]bgp 65000
[AR2-bgp]confederation id 100  //生成联盟号
[AR2-bgp]peer 10.1.23.3 as-number 65000
```

```bash
[AR3]bgp 65000
[AR3-bgp]confederation id 100  //必须先配置confederation
[AR3-bgp]confederation peer-as 65001  //指定联盟内的其他AS号
[AR3-bgp]peer 10.1.23.2 as-number 65000 
[AR3-bgp]peer 10.1.34.4 as-number 65001
```

```bash
[AR4]bgp 65001
[AR4-bgp]confederation id 100
[AR4-bgp]confederation peer-as 65000
[AR4-bgp]peer 10.1.34.3 as-number 65000
```

> - 在R2查看BGP邻居状态
> - 配置联盟必须先配置`confederation`，否则服务会起不来

## R1与R2配置BGP

- R1配置

```bash
[AR1]int lo 0
[AR1-LoopBack0]ip ad 1.1.1.1 32
[AR1-LoopBack0]bgp 200
[AR1-bgp]net 1.1.1.1 32 
[AR1]bgp 200
[AR1-bgp]peer 10.1.12.2 as-number 100
```

```bash
[AR2-bgp]peer 10.1.12.1 as-number 200
[AR2-bgp]peer 10.1.23.3 next-hop-local  //指定下一跳为本地
```

> - `refresh bgp all export`重新启动BGP进程
> - 在R2、R3查看BGP路由表
> - 在R3上查看的路由表`(65000)200i`表示经过了65000区域，起源于200

**AS-Path**

- AS-Path从右往左加
- AS-Path四种类型：

| 数值 | 类型 | 说明 |
|:---:|:---|:-----|
| 1 | AS_SET | 无序 |
| 2 | AS_SEQUENCE |有序排列 |
| 3 | AS_CONFED_SEQUENCE |  |
| 4 | AS_CONFED_SET |  |

# 总结

**路由反射器与联盟对比**

| 路由反射器 | 联盟 |
|:---|:---|
| 不需要更改现有的网络拓扑，兼容性好 | 需要改变逻辑拓扑 |
| 配置方便，只需要对作为反射器的设备进行配置，客户机并不需要知道自己是客户机 | 所有设备需要重新进行配置 |
| 集群与集群之间仍然需要全连接 | 联盟的子AS之间说特殊的EBGP连接，不需要全连接 |
| 适用于中、大规模网络 | 适用于大规模网络 |