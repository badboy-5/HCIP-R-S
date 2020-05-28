---
姓名：坏坏
实验时间：2020年5月28日
整理时间：2020年5月28日
---

# IS-IS练习实验

如下拓扑：

![IS-IS练习实验](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CIS-IS%E7%BB%83%E4%B9%A0%5CIS-IS%E7%BB%83%E4%B9%A0.assets%5CIS-IS%E7%BB%83%E4%B9%A0%E5%AE%9E%E9%AA%8C.png)

**实验需求**：

1. 按照图示配置IP地址
2. 按照图示分区域配置IS-IS，完成全网互通，网段192.168.x.1/24暂不宣告
3. R1、R2、R3 共享网络中，要求R3成为DIS，但只允许在R1与R2上配置
4. R4与R6和R5与 R6 之间不允许有DIS路由器选举
5. 按图示修改各链路 cost，只针对level-2进level-1路由修改开销
6. R6引入192.168.x.1/24网段路由，并标记为100
7. 要求区域 49.0001 能够学习到192.168.x.1/24 明细路由，并能看到标记100，并选择最优路由
8. 要求R4与R5只形成level-1的邻居关系
9. 在骨干网上配置接口认证

## 基础配置

- 配置IP地址

```bash
-----------------------------------------
# AR1基础配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.1 255.255.255.0 
-----------------------------------------
# AR2基础配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.2 255.255.255.0
-----------------------------------------
# AR3基础配置
 sysname AR3
#
interface Serial1/0/0
 link-protocol ppp
 ip address 34.1.1.3 255.255.255.0 
#
interface Serial1/0/1
 link-protocol ppp
 ip address 35.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.3 255.255.255.0
-----------------------------------------
# AR4基础配置
 sysname AR4
#
interface Serial1/0/0
 link-protocol ppp
 ip address 34.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 46.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 45.1.1.4 255.255.255.0
-----------------------------------------
# AR5基础配置
 sysname AR5
#
interface Serial1/0/0
 link-protocol ppp
 ip address 35.1.1.5 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 56.1.1.5 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 45.1.1.5 255.255.255.0 
-----------------------------------------
# AR6基础配置
interface GigabitEthernet0/0/0
 ip address 46.1.1.6 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 56.1.1.6 255.255.255.0 
#
interface LoopBack0
 ip address 192.168.0.1 255.255.255.0 
#
interface LoopBack1
 ip address 192.168.1.1 255.255.255.0 
#
interface LoopBack2
 ip address 192.168.2.1 255.255.255.0 
#
interface LoopBack3
 ip address 192.168.3.1 255.255.255.0 
```

## ISIS配置

- 配置IS-IS实现全网互通

```bash
# AR1配置
[AR1]isis 1
[AR1-isis-1]is-level level-1
[AR1-isis-1]net 49.0001.0000.0000.0001.00
[AR1-isis-1]int g0/0/0
[AR1-GigabitEthernet0/0/0]isis enable 1
```
```bash
# AR2配置
[AR2]isis 1
[AR2-isis-1]is-level level-1
[AR2-isis-1]net 49.0001.0000.0000.0002.00
[AR2-isis-1]int g0/0/0
[AR2-GigabitEthernet0/0/0]isis enable 1
```
```bash
# AR3配置
[AR3]isis 1
[AR3-isis-1]is-level level-1  
[AR3-isis-1]net 49.0001.0000.0000.0003.00
[AR3-isis-1]int g0/0/0
[AR3-GigabitEthernet0/0/0]isis enable 1
[AR3-GigabitEthernet0/0/0]int s1/0/0
[AR3-Serial1/0/0]isis enable 1
[AR3-Serial1/0/0]int s1/0/1
[AR3-Serial1/0/1]isis enable 1
```
```bash
# AR4配置
[AR4]isis 1
[AR4-isis-1]is-level level-1-2  //默认即为Level-1-2
[AR4-isis-1]net 49.0001.0000.0000.0004.00
[AR4-isis-1]int s1/0/0
[AR4-Serial1/0/0]isis enable 1
[AR4-Serial1/0/0]int g0/0/1
[AR4-GigabitEthernet0/0/1]isis enable 1
[AR4-GigabitEthernet0/0/1]int g0/0/0
[AR4-GigabitEthernet0/0/0]isis enable 1
```
```bash
# AR5配置
[AR5]isis 1
[AR5-isis-1]is-level level-1-2  
[AR5-isis-1]net 49.0001.0000.0000.0005.00
[AR5-isis-1]int s1/0/1
[AR5-Serial1/0/1]isis enable 1
[AR5-Serial1/0/1]int g0/0/0
[AR5-GigabitEthernet0/0/0]isis enable 1
[AR5-GigabitEthernet0/0/0]int g0/0/1
[AR5-GigabitEthernet0/0/1]isis enable 1
```
```bash
# AR6配置
[AR6]isis 1
[AR6-isis-1]is-level level-2
[AR6-isis-1]net 49.0002.0000.0000.0006.00
[AR6-isis-1]int g0/0/0
[AR6-GigabitEthernet0/0/0]isis enable 1 
[AR6-GigabitEthernet0/0/0]int g0/0/1
[AR6-GigabitEthernet0/0/1]isis enable 1
```

## 配置R3为DIS

- R1、R2、R3 共享网络中，要求 R3 成为 DIS，但只允许在 R1 与 R2 上配置
- IS-IS接口的默认优先级为64，是R3成为DIS，且在R1与R2上配置，则降低R1和R2的优先级

```bash
# AR1配置
[AR1]int g0/0/0 
[AR1-GigabitEthernet0/0/0]isis dis-priority 0
```
```bash
# AR2配置
[AR2]int g0/0/0
[AR2-GigabitEthernet0/0/0]isis dis-priority 0
```
```bash
# AR3查看
[AR3]dis isis int g0/0/0  //查看接口是否为DIS
```

- R4 与 R6 和 R5 与 R6 之间不允许有 DIS 路由器选举
- ISIS协议中，广播类型的网络中会有DIS设备选举
- 接口的网络类型为P2P时，不存在DIS设备选举

```bash
# AR4配置
[AR4]int g0/0/0
[AR4-GigabitEthernet0/0/0]isis circuit-type p2p
```
```bash
# AR5配置
[AR5]int g0/0/0
[AR5-GigabitEthernet0/0/0]isis circuit-type p2p 
```
```bash
# AR6配置
[AR6]int g0/0/0
[AR6-GigabitEthernet0/0/0]isis circuit-type p2p 
[AR6-GigabitEthernet0/0/0]int g0/0/1
[AR6-GigabitEthernet0/0/1]isis circuit-type p2p
```
```bash
# AR6查看
[AR6]dis isis int  //查看是否存在DIS
```

## 修改开销

- ISIS中，路由开销默认为10，只在进接口叠加链路开销

```bash
# AR4配置
[AR4]int s1/0/0
[AR4-Serial1/0/0]isis cost 5
```
```bash
# AR5配置
[AR5]int s1/0/0
[AR5-Serial1/0/0]isis cost 10 
```
```bash
# AR6配置
[AR6]int g0/0/0
[AR6-GigabitEthernet0/0/0]isis cost 50
[AR6-GigabitEthernet0/0/0]int g0/0/1
[AR6-GigabitEthernet0/0/1]isis cost 40
```

## 引入路由

- R6 引入 192.168.x.1/24 网段路由，并标记为 100

```bash
# AR6配置
[AR6]isis 1
[AR6-isis-1]import-route direct tag 100  //ISIS引入直连路由，并标记为100
[AR6-isis-1]cost-style wide  //修改路由计算模式
```

- 区域49.0001学习到192.168.x.1/24明细路由，并能看到标记100，并选择最优路由

```bash
# AR1配置
[AR1]isis 1
[AR1-isis-1]cost-style wide  //修改路由计算模式
```
```bash
# AR2配置
[AR2]isis 1
[AR2-isis-1]cost-style wide 
```
```bash
# AR3配置
[AR3]isis 1
[AR3-isis-1]cost-style wide
```
```bash
# AR4配置
[AR4]isis 1
[AR4-isis-1]cost-style wide
[AR4-isis-1]import-route isis level-2 into level-1 tag 100
```
```bash
# AR5配置
[AR5]isis 1
[AR5-isis-1]cost-style wide
[AR5-isis-1]import-route isis level-2 into level-1 tag 100
```

## 修改邻接关系

```bash
# AR4配置
[AR4]int g0/0/1
[AR4-GigabitEthernet0/0/1]isis circuit-level level-1

# AR5配置
[AR5]int g0/0/1
[AR5-GigabitEthernet0/0/1]isis circuit-level level-1 
```

## 骨干网络配置认证

- 连续的Level-2和Level-1-2路由器组成的网络位骨干网络，即AR4、AR5、AR6组成的网络

```bash
# AR4配置
[AR4]int g0/0/0
[AR4-GigabitEthernet0/0/0]isis authentication-mode simple plain bad-5
```
```bash
# AR5配置
[AR5]int g0/0/0
[AR5-GigabitEthernet0/0/0]isis authentication-mode simple plain bad-5
```
```bash
# AR6配置
[AR6]int g0/0/0
[AR6-GigabitEthernet0/0/0]isis authentication-mode simple plain bad-5
[AR6-GigabitEthernet0/0/0]int g0/0/1
[AR6-GigabitEthernet0/0/1]isis authentication-mode simple plain bad-5
```

> - 重启接口或重启IS-IS进程，IS-IS关系仍可以建立
> - 在AR1上可以查到AR6上的明细路由