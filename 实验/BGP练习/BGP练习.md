---
姓名：坏坏
实验时间：2020年6月1日
整理时间：2020年6月1日
---

# BGP练习实验

如下拓扑：

![BGP实验练习](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E7%BB%83%E4%B9%A0%5CBGP%E7%BB%83%E4%B9%A0.assets%5CBGP%E5%AE%9E%E9%AA%8C%E7%BB%83%E4%B9%A0.png)

**实验需求**：

1. 配置相应的接口IP地址，除AR3外，其他路由器配置环回口地址
2. 在AS 200区域内配置OSPF实现互通
3. 在AR2和AR4之间配置IBGP
4. 在AR1和AR2、AR4和AR5之间配置EBGP

	使AR1环回口地址可以与AR5环回口地址互通

## 基础配置

```bash
----------------------------------------------
# AR1基础配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 12.1.1.1 255.255.255.0
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
----------------------------------------------
# AR2基础配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 12.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 23.1.1.2 255.255.255.0
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255
----------------------------------------------
# AR3基础配置
 sysname AR3
#
interface GigabitEthernet0/0/0
 ip address 23.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 34.1.1.3 255.255.255.0
----------------------------------------------
# AR4基础配置
 sysname AR4
#
interface GigabitEthernet0/0/0
 ip address 34.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 45.1.1.4 255.255.255.0 
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255
----------------------------------------------
# AR5基础配置
 sysname AR5
#
interface GigabitEthernet0/0/0
 ip address 45.1.1.5 255.255.255.0
#
interface LoopBack0
 ip address 5.5.5.5 255.255.255.255
----------------------------------------------
```

## 配置OSPF

- AR2配置OSPF

```bash
[AR2]ospf           
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
```

- AR3配置OSPF

```bash
[AR3]ospf 
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255
```

- AR4配置OSPF

```bash
[AR4]ospf
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0
[AR4-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255
```

## 配置IBGP

- AR2配置

```bash
[AR2]bgp 200  //AS区域为200
[AR2-bgp]router-id 2.2.2.2  //指定Router-ID
[AR2-bgp]peer 4.4.4.4 as-number 200  //指定邻居的环回口地址及所属区域
[AR2-bgp]peer 4.4.4.4 connect-interface LoopBack 0  //指定更新源
```

- AR4配置

```bash
[AR4]bgp 200
[AR4-bgp]router-id 4.4.4.4
[AR4-bgp]peer 2.2.2.2 as-number 200
[AR4-bgp]peer 2.2.2.2 connect-interface LoopBack 0
```

- 查看邻居建立

```bash
[AR2]dis bgp peer  //查看邻居建立
```

> - 在IBGP中，必须指定更新源接口，一般使用环回口
> - BGP邻居建立状态为`Active`没有建立成功，`Established`表示邻居建立成功

## 配置EBGP

- AR1配置EBGP

```bash
[AR1]bgp 100
[AR1-bgp]peer 12.1.1.2 as-number 200  //指定对端的直连接口和对端的AS区域
```

- AR2配置EBGP

```bash
[AR2]bgp 200
[AR2-bgp]peer 12.1.1.1 as-number 100
```

- AR4配置EBGP

```bash
[AR4]bgp 200
[AR4-bgp]peer 45.1.1.5 as-number 300
```

- AR5配置EBGP

```bash
[AR5]bgp 300
[AR5-bgp]peer 45.1.1.4 as-number 200
```

> - 在AR2和AR3上查看BGP邻居建立关系

## 将路由发布到BGP

- AR1发布路由

```bash
[AR1]bgp 100
[AR1-bgp]network 1.1.1.1 32
```

- AR5发布路由

```bash
[AR5]bgp 300
[AR5-bgp]net 5.5.5.5 32
```

> - 发布的路由必须与本地的路由表中的路由保持一致
> - AR1发布路由后，可以在AR2、AR3查看是否得到路由
> 	* `dis bgp routing-table`查看协议路由表
> - 通过查看协议路由表，看到路由前的`*>`
> 	* `*`表示此路由可用
> 	* `>`表示此路由为最优
> - 一条路由不可用时，不会将此路由转发给邻居

## 配置路由

- 配置完以上步骤，查看BGP协议路由会发现路由不可用，因为下一跳不可达，需要配置路由

- AR2配置路由（12.1.1.0宣告进OSPF），使下一跳可达

```bash
[AR2]ospf
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 12.1.1.0 0.0.0.255
```

- 修改BGP中路由的下一跳

```bash
[AR4]bgp 200
[AR4-bgp]peer 2.2.2.2 next-hop-local  //修改到达邻居的下一跳
```

> - 在AR1、AR5查看BGP协议的路由表

## AR2配置路由

- **路由黑洞**：中间设备没有路由，数据到中间设备后无法转发

- 在AR2、AR4的OSPF中引入BGP路由

```bash
# AR2上引入BGP
[AR2]ospf
[AR2-ospf-1]import-route bgp 

# AR4上引入OSPF
[AR4]ospf
[AR4-ospf-1]import-route bgp 
```

> - 也可以使用MPLS、VPN等方法引入BGP路由
> - BGP路由一般不引入IGP路由