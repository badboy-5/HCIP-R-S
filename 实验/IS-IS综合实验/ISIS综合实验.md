# 如下拓扑：

![1590548021296](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CIS-IS%5CISIS%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C1590548021296.png)

# 实验需求：


1. 客户网络所有路由器路由协议要求启用IS-IS，使全网路由可达。
2. 全部IS-IS进程号统一为100，其中AR1在Area 49.0001区域为DIS，AR4与AR5之间要求采用P2P网络类型
3. AR5引入直连链路192.168.X.X，要求AR1访问Area49.0002走最优路径。

	根据上述描述，进行正确配置，使网络路由达到客户需求

# 配置IP地址

```bash
------------------------------------------------
# AR1配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.1 255.255.255.0 
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
------------------------------------------------
# AR2配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 24.1.1.2 255.255.255.0 
------------------------------------------------
# AR3配置
 sysname AR3
#
interface Serial1/0/0
 link-protocol ppp
 ip address 34.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 123.1.1.3 255.255.255.0
------------------------------------------------
# AR4配置
 sysname AR4
#
interface Serial1/0/0
 link-protocol ppp
 ip address 34.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/0
 ip address 24.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 45.1.1.4 255.255.255.0 
------------------------------------------------
# AR5配置
 sysname AR5
#
interface GigabitEthernet0/0/0
 ip address 45.1.1.5 255.255.255.0 
#
interface LoopBack0
 ip address 5.5.5.5 255.255.255.255 
#
interface LoopBack1
 ip address 192.168.1.1 255.255.255.255 
#
interface LoopBack2
 ip address 192.168.2.1 255.255.255.255 
#
interface LoopBack3
 ip address 192.168.3.1 255.255.255.255
```

# 配置基础IS-IS

- AR1为Level-1，AR2、AR3为Level-1-2（默认），AR4、AR5为Level-2

```bash
# AR1配置
[AR1]isis 100  //ISIS进程号
[AR1-isis-100]is-level level-1  //配置为Level-1
[AR1-isis-100]network-entity 49.0001.0000.0000.0001.00
[AR1-isis-100]int g0/0/0
[AR1-GigabitEthernet0/0/0]isis enable 100
[AR1-GigabitEthernet0/0/0]int lo 0
[AR1-LoopBack0]isis enable 100

# AR2配置
[AR2]isis 100
[AR2-isis-100]network-entity 49.0001.0000.0000.0002.00
[AR2-isis-100]int g0/0/0
[AR2-GigabitEthernet0/0/0]isis enable 100
[AR2-isis-100]int g0/0/1
[AR2-GigabitEthernet0/0/1]isis enable 100

# AR3配置
[AR3]isis 100
[AR3-isis-100]network-entity 49.0001.0000.0000.0003.00
[AR3-isis-100]int g0/0/0
[AR3-GigabitEthernet0/0/0]isis enable 100
[AR3-GigabitEthernet0/0/0]int s1/0/0
[AR3-Serial1/0/0]isis enable 100

# AR4配置
[AR4]isis 100
[AR4-isis-100]is-level level-2
[AR4-isis-100]network-entity 49.0002.0000.0000.0004.00
[AR4-isis-100]int g0/0/0
[AR4-GigabitEthernet0/0/0]isis enable 100
[AR4-GigabitEthernet0/0/0]int s1/0/0  
[AR4-Serial1/0/0]isis enable 100
[AR4-Serial1/0/0]int g0/0/2
[AR4-GigabitEthernet0/0/2]isis enable 100

# AR5配置
[AR5]isis 100
[AR5-isis-100]is-level level-2
[AR5-isis-100]net 49.0002.0000.0000.0005.00
[AR5-isis-100]int g0/0/0
[AR5-GigabitEthernet0/0/0]isis enable 100
[AR5-GigabitEthernet0/0/0]int lo 0
[AR5-LoopBack0]isis enable 100
```


# 详细配置

```bash
# 修改优先级,是AR1成为DIS
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]isis dis-priority 70

[AR1]dis isis interface g0/0/0  //查看ISIS状态，AR1为Level-1的DIS

# 链路类型修改为P2P
[AR4]int g0/0/2
[AR4-GigabitEthernet0/0/2]isis circuit-type p2p

# AR5修改链路类型为P2P
[AR5]int g0/0/0
[AR5-GigabitEthernet0/0/0]isis circuit-type p2p
```

	P2P两端的链路类型都需要修改

- Level-1引入Level-2,并配置最优路径

```bash
# AR5引入直连路由
[AR5]isis 100
[AR5-isis-100]import-route direct  //引入直连路由

# AR2、AR3把Level-2引入Level-1
[AR2]isis 100
[AR2-isis-100]import-route isis level-2 into level-1 

[AR3]isis 100
[AR3-isis-100]import-route isis level-2 into level-1 

# 修改AR2的开销配置最优路径
[AR2]int g0/0/1
[AR2-GigabitEthernet0/0/1]isis cost 1  //修改开销，ISIS中默认为10

[AR1]dis ip routing-table  //查看路由表信息
```

> 在修改开销之前，AR1访问Area 49.0002通过AR2和AR3负载，修改将AR2的出接口的开销改小，数据会优先从AR2转发
> AR2上的接口down后，数据走AR3，恢复后继续从AR2转发