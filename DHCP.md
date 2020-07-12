---
姓名：坏坏
学习时间：2020年7月1日
整理时间：2020年7月1日
---

# DHCP

## DHCP工作过程

**DHCP请求地址**

![DHCP请求地址](F:\GitHub\HCIP R&S\DHCP.assets\DHCP请求地址.png)

- **发现阶段**：客户端通过广播方式发送DHCP Discover消息，寻找DHCP服务器，表示自己需要一个IP地址
- **提供阶段**：DHCP服务器收到DHCP Discover消息后，把可以提供的IP地址携带在DHCP Offer中，单播发送给客户端
- **请求阶段**：客户端收到DHCP Offer后，选择第一个到达的Offer，并通过广播方式向相应的DHCP服务器发送DHCP Request消息，请求使用这个IP地址
- **确认阶段**：DHCP服务器收到DHCP Request消息后，该IP地址如果可以使用，会单播回复DHCP Ack消息，表示地址分配成功。如果不可以使用，会单播回复DHCP Nak消息，表示地址分配失

> - 发现阶段客户端也可以通过单播、组播方式发送DHCP Discover消息
> - 请求阶段客户端广播发送DHCP Request消息还可以使其他的HDCP服务器释放已经准备分配给客户端的IP地址
> - 思科设备全部是广播方式发送报文，华为设备是广播—单播—广播—单播

**DHCP地址续租**

![DHCP续租](F:\GitHub\HCIP R&S\DHCP.assets\DHCP续租.png)

- 当租期约过了1/2时，客户端向DHCP服务器单播发送DHCP Ruquest消息，希望续租IP地址
- 服务器收到后，会刷新客户端的地址租期时间，重新计时，并发送DHCP Ack消息
- 如果DHCP服务器没有回复，客户端会等租期过了7/8（87.5%）时，还没有收到DHCP服务器的回复，会再次广播发送DHCP Request消息，向所有的DHCP服务器请求续租IP地址
- 在租期到期前，收到了DHCP Ack消息，则续租成功，可以继续使用该IP地址。如果没有收到消息，则续租失败，不能继续使用该IP地址

## DHCP配置

**基于全局地址池**

```bash
dhcp enable
ip pool HW
 gateway-list 192.168.1.1 
 network 192.168.1.0 mask 255.255.255.0 
 excluded-ip-address 192.168.1.2 
 lease day 3 hour 0 minute 0 
 dns-list 192.168.1.2 
interface GigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.0 
 dhcp select global
```

**基于接口**

```bash
dhcp enable
interface g0/0/0
 ip address 192.168.1.1 24
 dhcp select interface
 dhcp server dns-list 192.168.1.2
 dhcp server excluded-ip-address 192.168.1.2
 dhcp server lease day 2 hour 0 minute 0 
```

## DHCP Relay

![DHCP Relay](F:\GitHub\HCIP R&S\DHCP.assets\DHCP Relay.png)

- 客户端广播发送DHCP Discover消息，DHCP Relay收到消息后，向DHCP服务器单播发送DHCP Discover消息
- 服务器收到Discover消息后，单播回复DHCP Offer，DHCP Relay收到后再单播发送给客户端
- 客户端再广播发送DHCP Request消息，DHCP Relay收到后单播发送给DHCP服务器
- 如果地址可用，DHCP服务器会单播回复DHCP Ack，不可用会单播回复DHCP Nak，DHCP Relay收到消息后，再单播给客户端


## 实验

如下拓扑图：

![DHCP Relay实验拓扑](F:\GitHub\HCIP R&S\DHCP.assets\DHCP Relay实验拓扑.png)

- 基本配置

```bash
#
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.1 255.255.255.0 
-----------------------------------------------
#
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 192.168.1.254 255.255.255.0 
```

- 配置全局地址池

```bash
[AR1]ip pool bad  //创建全局地址池
[AR1-ip-pool-bad]network 192.168.1.0 mask 24  //地址池可以分配的地址
[AR1-ip-pool-bad]gateway-list 192.168.1.254  //网关
[AR1-ip-pool-bad]dns-list 1.1.1.1  //DNS
[AR1-ip-pool-bad]excluded-ip-address 192.168.1.10 192.168.1.30  //排除的IP地址
[AR1-ip-pool-bad]lease day 5 hour 15 minute 20  //租期
[AR1-ip-pool-bad]q
[AR1]dhcp enable  //全局开启DHCP服务
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]dhcp select global  //接口下选择全局地址池
```

- 配置DHCP中继

```bash
[AR2]dhcp enable 
[AR2]int g0/0/1
[AR2-GigabitEthernet0/0/1]dhcp relay server-ip 10.1.12.1
```

- 配置静态路由

```bash
[AR1]ip route-static 192.168.1.0 24 10.1.12.2
```

- 抓包分析

![抓包分析1](F:\GitHub\HCIP R&S\DHCP.assets\抓包分析1.png)

![抓包分析2](F:\GitHub\HCIP R&S\DHCP.assets\抓包分析2.png)

> PC上使用DHCP获取IP地址，查看IP地址、网关、DNS等信息

- 路由器配置获得IP

```bash
[AR3]dhcp enable  //全局开启DHCP
[AR3]int g0/0/0
[AR3-GigabitEthernet0/0/0]ip ad dhcp-alloc  //DHCP分配地址
[AR3-GigabitEthernet0/0/0]dis ip int bri  //查看接口IP
```

> 不仅PC可以通过DHCP获得IP地址，路由器也可以通过DHCP获得IP地址

## DHCP饿死攻击

- 攻击原理：攻击者持续大量的向DHCP Server申请IP地址，直到耗尽DHCP Server地址池中的IP地址，导致DHCP Server不能对正常的用户进行分配
- 漏洞分析：DHCP Server向申请者分配IP地址时，无法区分是否为正常的申请

> 攻击者通过不断修改DHCP报文中的CHADDR字段值，持续不断的向DHCP Server申请IP地址

## 仿冒DHCP Server攻击

- 攻击原理：攻击者仿冒DHCP Server，向客户端分配错的IP地址及提供错误的网关地址等参数，导致客户端无法正常访问网络
- 漏洞分析：DHCP客户端接收到来自DHCP Server的DHCP消息后，无法区分这些DHCP消息是来自仿冒的DHCP Server还是来自合法的DHCP Server

> 客户端广播发送DHCP Discover消息，仿冒DHCP Server尽快恢复DHCP Offer报文，使客户端收到错误的IP地址、网关等信息

## DHCP中间人攻击

- 攻击原理：攻击者利用ARP机制，让PC学习到服务器IP与攻击者MAC的映射关系，又让服务器学习到PC的IP地址与攻击者MAC的映射关系，则PC与服务器之间交互的IP报文都会经过攻击者中转
- 漏洞分析：本质上，中间人攻击是一种Spoofing　IP/MAC攻击，中间人利用了虚假的IP地址与MAC地址之间的映射关系来同时欺骗DHCP的客户端和服务器

## DHCP Snooping

- DHCP Snooping用于防止DHCP饿死攻击
	* DHCP Snooping支持在端口下对DHCP Request报文的源MAC地址和CHADDR进行一致性检查，相同则转发，不同则丢弃
	* 配置命令：`dhcp snooping check dhcp-chaddr enable`（接口视图下）
- DHCP Snooping用于防止DHCP Server攻击
	* DHCP Snooping将交换机上的端口分为信任端口（Trusted）和非信任端口（Untrusted）
	* 与合法DHCP Server相连的端口配置为Trusted端口，其他端口配置为Untrusted端口
	* 交换机从Trusted端口接收到的DHCP Offer、DHCP Ack/Nak会进行转发，从Untrusted端口接收到的相关报文则直接丢弃
	* 配置命令：`dhcp snooping trusted`(默认端口为Untrusted)（接口视图）
- DHCP Snooping用于防止DHCP中间人攻击
	* DHCP Snooping会侦听DHCP Ack消息，将收集到的用户MAC地址，DHCP Server分配给用户的IP地址存放到DHCP Snooping绑定表，表项中包含用户的MAC地址、IP地址、IP地址租期、VLAN-ID等信息的对应关系
	* 攻击者发送ARP请求报文，将源IP地址更改为用户端的IP地址，MAC地址为自身的MAC地址。交换机收到ARP请求后，检查ARP报文中的源IP、MAC，发现于DHCP Snooping绑定表种的表项不一致，则会丢弃该ARP请求
	* 配置命令：`arp dhcp-snooping-detect enable`（交换机的系统视图下）

## DHCP Snooping与IPSG技术联动

- IPSG（Source Guard）IP源防护，
- 交换机启用IPSG功能后，会对进入交换机端口的报文进行合法性检查，并对报文进行过滤，合法则转发，不合法则丢弃
- 在交换机的端口视图下：
	* IP+MAC
	* IP+VLAN
	* IP+MAC+VLAN
- 在交换机VLAN视图下：
	* IP+MAC
	* IP+物理端口号
	* IP+MAC+物理端口号
- 配置命令：`ip source check user-bind enable`

> DHCP Snooping与IPSG技术联动可以只检测IP地址或MAC地址等，没有IPSG则会把DHCP Snooping绑定表中的每一项都检查
