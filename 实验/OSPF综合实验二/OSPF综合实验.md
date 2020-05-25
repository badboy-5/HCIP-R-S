---
姓名：坏坏
实验时间：2020年5月25日
整理时间：2020年5月25日
---

# OSPF综合实验

**声明**：本实验参考拓扑来自[【王文杰在线文档库】](http://www.wwenjie.com/15439706659130.html)

如下拓扑图：

![1590402508742](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5COSPF%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C%E4%BA%8C%5COSPF%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C1590402508742.png)

**实验需求**：

按照图示配置 IP 地址
	1. R2，R3，R4 运行 OSPF ，所有接口（公网接口除外）全部宣告进 Area 0；R4，R5 运行 OSPF ，所有接口全部宣告进 Area 1；要求使用环回口作为 Router-id ；ABR 环回口须宣告进骨干区域
	2. R2 模拟互联网，内网通过 R2 连接互联网，在 R2 上配置默认路由并引入到 OSPF
	3. R2 上配置 EASY IP，只允许业务网段访问互联网
	4. 业务网段不允许出现协议报文，要求业务网段访问互联网流量经过R4与R2时保证负载效果，并保证来回路径一致

## 基础配置

```bash
----------------------------------------------
# AR1基础配置
interface GigabitEthernet0/0/0
 ip address 12.1.1.1 255.255.255.0 
#
interface LoopBack0
 ip address 100.1.1.1 255.255.255.255 
-----------------------------------------------
# AR2基础配置
interface GigabitEthernet0/0/0
 ip address 12.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 23.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 24.1.1.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255
-----------------------------------------------
# AR3基础配置
interface GigabitEthernet0/0/0
 ip address 23.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 34.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/2
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255 
-----------------------------------------------
# AR4基础配置
interface GigabitEthernet0/0/0
 ip address 34.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 24.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 45.1.1.4 255.255.255.0 
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255 
-----------------------------------------------
# AR5基础配置
interface GigabitEthernet0/0/0
 ip address 45.1.1.5 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 192.168.1.254 255.255.255.0 
#
interface LoopBack0
 ip address 5.5.5.5 255.255.255.255
```

## 配置OSPF

```bash
# AR2配置
[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255

# AR3配置
[AR3]ospf router-id 3.3.3.3
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255

# AR4配置
[AR4]ospf router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0
[AR4-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255
[AR4-ospf-1]area 1
[AR4-ospf-1-area-0.0.0.1]net 45.1.1.0 0.0.0.255

# AR5配置
[AR5]ospf router-id 5.5.5.5
[AR5-ospf-1]area 1
[AR5-ospf-1-area-0.0.0.1]net 5.5.5.5 0.0.0.0
[AR5-ospf-1-area-0.0.0.1]net 45.1.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.1]net 192.168.1.0 0.0.0.255
```

## 配置并引入默认路由

```bash
[AR2]ip route-static 0.0.0.0 0 12.1.1.1  //配置默认路由
[AR2]ospf 1
[AR2-ospf-1]default-route-advertise  //引入默认路由
```

## 配置Easy IP

```bash
[AR2]acl 2000  //配置ACL
[AR2-acl-basic-2000]rule 5 permit source 192.168.1.0 0.0.0.255  //允许业务网段
[AR2-acl-basic-2000]q
[AR2]int g0/0/0
[AR2-GigabitEthernet0/0/0]nat outbound 2000  //出接口调用ACL
```

	PC与AR1环回口连通性测试

## 配置静默接口并设置负载均衡

```bash
[AR5]ospf 
[AR5-ospf-1]silent-interface g0/0/1  //设置静默接口

[AR2]int g0/0/2
[AR2-GigabitEthernet0/0/2]ospf cost 2  //修改开销

[AR4]int g0/0/1
[AR4-GigabitEthernet0/0/1]ospf cost 2
```

	查看AR4和AR2的路由表，默认路由和192.168.1.0为负载

