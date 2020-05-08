---
姓名：王杰
学习时间：2020年3月15日
整理时间：2020年5月7日
---

# VRRP协议

- 一种网关备份协议
	* 实现网关的备份
	* 解决多个网关之间互相冲突的问题

## 单网关的缺陷

- 当网关路由器出现故障时，本网段以该设备为网关的主机都不能与Internet进行通信

## 多网关存在的问题

- 网关间IP地址冲突
- 主机会频繁切换网络出口

# VRRP基本概述

- VRRP可以在不改变组网的情况下，将多台路由器虚拟成一个虚拟路由器，通过配置虚拟路由器的IP地址为默认网关，实现网关的备份
- 协议版本
	* **VRRPv2**（常用），仅适用于IPv4网络
	* **VRRPv3**适用于IPv4和IPv6两种网络
- VRRP协议报文
	* 只有一种报文（Advertisement报文）
	* 目的IP地址224.0.0.18，目的MAC地址01-00-5e-00-00-12，协议号112

# VRRP基本结构

- 虚拟路由器中有Master和Backup，当Master发生故障之后，启用Backup设备
- 虚拟路由器的虚拟MAC为：0000-5e00-01+VRID

# VRRP主备备份工作过程

- 路由器交互报文，选举优先级大的设备作为Master，小的为Backup
- 100为VRRP的默认优先级
- Master周期性（1s一次）的发送通告报文给组内其他设备，通知自己处于正常工作状态
- VRRP主备选完之后，主设备会向外发送免费ARP，可以检测IP地址冲突。在VRRP中，中间设备或PC会把免费ARP的源IP、源Mac写入ARP表

## Master发生故障

- 发生以下故障Backup会抢占Master
	* Backup收不到Master发送的状态报文
	* 通告报文中的优先级比Backup小
- 解决方法：
	* 利用VRRP的联动功能监视上行接口或链路故障，主动进行主备切换

# 实验

如下拓扑，配置VRRP

![](F:%5CGitHub%5CHCIP%20R&S%5CVRRP.assets%5CVRRP%E5%AE%9E%E9%AA%8C%E6%8B%93%E6%89%91.png)

- 配置接口IP地址

```bash
[AR1]int g0/0/1
[AR1-GigabitEthernet0/0/1]ip ad 13.1.1.1 24
[AR1-GigabitEthernet0/0/1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip ad 192.168.1.3 24

[AR2]int g0/0/1
[AR2-GigabitEthernet0/0/1]ip ad 23.1.1.2 24
[AR2-GigabitEthernet0/0/1]int g0/0/0
[AR2-GigabitEthernet0/0/0]ip ad 192.168.1.4 24

[AR3]int g0/0/0
[AR3-GigabitEthernet0/0/0]ip ad 13.1.1.3 24
[AR3-GigabitEthernet0/0/0]int g0/0/1
[AR3-GigabitEthernet0/0/1]ip ad 23.1.1.3 24
[AR3-GigabitEthernet0/0/1]int lo 0
[AR3-LoopBack0]ip ad 3.3.3.3 32
```

- 配置OSPF互联

```bash
[AR1]ospf
[AR1-ospf-1]area 0
[AR1-ospf-1-area-0.0.0.0]net 13.1.1.0 0.0.0.255
[AR1-ospf-1-area-0.0.0.0]net 192.168.1.0 0.0.0.255

[AR2]ospf 
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 192.168.1.0 0.0.0.255

[AR3]ospf
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 13.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
```

- 配置VRRP

```bash
# R1配置，使AR1成为VRID 1中的Master
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]vrrp vrid 1 virtual-ip 192.168.1.254  //配置VRID 1，配置虚拟网关
[AR1-GigabitEthernet0/0/0]vrrp vrid 2 virtual-ip 192.168.1.253  //配置VRID 2
[AR1-GigabitEthernet0/0/0]vrrp vrid 2 priority 120  //更改VRID 2中AR1的优先级

[AR1]dis vrrp
  GigabitEthernet0/0/0 | Virtual Router 1
    State : Backup
    Virtual IP : 192.168.1.254
    Master IP : 192.168.1.4
    PriorityRun : 100
    PriorityConfig : 100
    MasterPriority : 120
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE
    Virtual MAC : 0000-5e00-0101
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Create time : 2020-05-07 10:16:59 UTC-08:00
    Last change time : 2020-05-07 11:14:17 UTC-08:00

  GigabitEthernet0/0/0 | Virtual Router 2
    State : Master
    Virtual IP : 192.168.1.253
    Master IP : 192.168.1.3
    PriorityRun : 120
    PriorityConfig : 120
    MasterPriority : 120
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE
    Virtual MAC : 0000-5e00-0102
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Create time : 2020-05-07 10:46:46 UTC-08:00
    Last change time : 2020-05-07 11:13:55 UTC-08:00

[AR1]

# R2配置VRID 2，是AR2成为VRID 2中的Master
[AR2]int g0/0/0
[AR2-GigabitEthernet0/0/0]vrrp vrid 1 virtual-ip 192.168.1.254
[AR2-GigabitEthernet0/0/0]vrrp vrid 2 virtual-ip 192.168.1.253
[AR2-GigabitEthernet0/0/0]vrrp vrid 1 priority 120  //更改VRID 1中AR2的优先级

[AR2]dis vrrp
  GigabitEthernet0/0/0 | Virtual Router 1
    State : Master
    Virtual IP : 192.168.1.254
    Master IP : 192.168.1.4
    PriorityRun : 120
    PriorityConfig : 120
    MasterPriority : 120
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE
    Virtual MAC : 0000-5e00-0101
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Create time : 2020-05-07 10:17:54 UTC-08:00
    Last change time : 2020-05-07 11:14:17 UTC-08:00

  GigabitEthernet0/0/0 | Virtual Router 2
    State : Backup
    Virtual IP : 192.168.1.253
    Master IP : 192.168.1.3
    PriorityRun : 100
    PriorityConfig : 100
    MasterPriority : 120
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE
    Virtual MAC : 0000-5e00-0102
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Create time : 2020-05-07 10:45:50 UTC-08:00
    Last change time : 2020-05-07 11:13:55 UTC-08:00

[AR2]
```

> - PC1网关为：192.168.1.254
> - PC2网关为：192.168.1.253

- 抓包查看

![1588824953910](F:%5CGitHub%5CHCIP%20R&S%5CVRRP.assets%5C1588824953910.png)

- PC端ping3.3.3.3测试

![1588825242686](F:%5CGitHub%5CHCIP%20R&S%5CVRRP.assets%5C1588825242686.png)

> - 可以看到PC1在ping3.3.3.3时，是通过R2，但是只有Request请求，没有Reply回应，但还是通了，因为回应报文是从R1走的
> - PC2在ping3.3.3.3时，是通过R1转发的

## 下行接口、AR1出现故障

- VRID 2中，当AR1G0/0/0接口down或AR1出现故障，会发生主备切换

**实验**：使用PC2 ping 3.3.3.3，然后关闭AR1的G0/0/0接口，查看PC端丢包情况

```bash
# PC端
PC>ping 3.3.3.3 -t  //一直ping 3.3.3.3

# AR1把G0/0/0口down掉
[AR1]int g0/0/0	
[AR1-GigabitEthernet0/0/0]shutdown 

# 再把G0/0/0接口UP
[AR1-GigabitEthernet0/0/0]undo shutdown
```

- 在PC端可以看到，当接口down掉之后，然后丢失了两个包，随即连通性恢复。在接口up之后，虽然发生了丢包，但之后依旧恢复了连通性

![1588927334814](F:%5CGitHub%5CHCIP%20R&S%5CVRRP.assets%5C1588927334814.png)

## 上行接口down

```bash
# 配置接口为静默接口
[AR1]ospf 
[AR1-ospf-1]silent-interface GigabitEthernet 0/0/0  //将G0/0/0设置为静默接口

[AR2]ospf
[AR2-ospf-1]silent-interface GigabitEthernet 0/0/0
```

- AR1的上行接口down，PC端ping 3.3.3.3，无法ping通，但是在AR1 的G0/0/0口抓包还是可以接收到Request报文

- 配置上行端口检测

```bash
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]vrrp vrid 2 track interface GigabitEthernet0/0/1 reduced 30  //上行端口down后，VRID 2的优先级降30
```

> 使AR1上行端口down后，查看VRRP的状态，就可以看到AR1上VRID 2的优先级降到了90，成为Backup，当上行接up后，重新变回120，成为Master

- 配置VRRP认证

```bash
[AR1-GigabitEthernet0/0/0]vrrp vrid 1 authentication-mode md5 huawei
[AR2-GigabitEthernet0/0/0]vrrp vrid 1 authentication-mode md5 huawei
```

> 当只有单边做VRRP认证或认证失败时，会出现双主的情况，需要在另一边也做认证